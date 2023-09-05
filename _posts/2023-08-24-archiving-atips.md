---

title: Archiving access to information and freedom of information requests

author: jamie_duncan

date: 2023-08-24

tags: [data, methods]

---
# Introduction
Access to Information and Freedom of Information (ATI/FOI) requests are valuable sources of data for academic, journalistic, and advocacy purposes. The laws that entitle citizens (and in many cases non-citizens) to information about how governments and public institutions work are under-appreciated democratic safeguards. This blog post offers a step-by-step guide to the workflow I developed using the R programming language to manage these files, link them with relevant metadata, and bulk upload them to the Internet Archive.
## Problems
ATI/FOI is just one tool in a much broader landscape of Open Government tools and policies – and it is far from perfect. This post addresses three challenges I experience in the Canadian context:
1. Documents released by the federal government are part of the public domain but are not proactively published. Instead, users can request previously released information based on [published request summaries](https://open.canada.ca/en/search/ati). Some people still publish copies of the documents they use in their research but these documents are strewn across the Internet and difficult to locate.
2. Citation practices related to ATI/FOI are inconsistent. While this may seem like a niche academic concern, it is relevant to anyone producing knowledge for public consumption. Citing sources is important so that others can verify the accuracy of claims they come across but also so that they can find information that helps them answer their own questions.
3. Documents are frequently released in non-machine-readable formats (typically .pdfs), which makes it inefficient to search for information in releases that frequently include hundreds of pages, and across large numbers of documents.
While ultimately these issues (and many others) should be addressed through changes to how the ATI/FOI system works, I am trying to mitigate some of them in the course of my work.
## Proposed 'solution'
The Canadian federal government receives over 200,000 ATI requests every year so it is not feasible for academics or civil society to take on the responsibility of obtaining and publishing all of these requests. However, what is feasible is to publish the documents I obtain for my own research to a single, stable repository: [The Internet Archive](https://archive.org/) (IA). IA has the added benefit of converting .pdf files to a machine readable format using Optical Character Recognition (OCR). It also allows you to tie each document to metadata like when it was released, the request summary, and the department that released it – all the things that someone citing the document would need!

What follows is a step-by-step guide to archiving ATIs (or whatever other documents) using R.
# Step-by-step guide to archiving ATI/FOI documents

I study borders and immigration. A while back I asked Immigration Refugees and Citizenship Canada (IRCC) for around 50 previously completed ATIs. These records will serve as the example in this guide. I have refined this method by archiving over 1500 documents from several other agencies that were obtained by me or my colleagues at the [Centre for Access to Information and Justice](https://www.uwinnipeg.ca/caij/). This guide presumes you have a folder of requests with the original file names. IRCC released their requests to me via email. I manually downloaded all the files and put them in my folder.

  The code in this post its just what I have hacked together. There are almost certainly more efficient and elegant ways of achieving the same outcomes. If you make something better, please share it!
  {: .prompt-warning}

## Clean names
The first thing we'll want to do is standardize the file names. Documents typically (but not always) feature a request number (like A-2020-06759). This number is not always formatted the same way and the document titles often include other information we don't need. For example, IRCC's releases included the following titles:
- 2A-2020-38873.pdf
- 2A202171766_Finalrelease.pdf
- 2A202016275_2020-08-06_13-31-42.pdf
- A202009593_ release pckg.pdf

The goal of cleaning the names is to ensure that all files have consistent file names that will serve as unique identifiers. We see already that the request number includes the year and (typically) a 5 digit number. But the request number alone is not a unique identifier. I've found different government departments publish documents using the same request number in a given year (the last five digits seem to be sequential). With that in mind I have chosen a naming convention which lists the agency followed by the request number. For example: "IRCC_A-2020-38873.pdf".

To transform the file names, I created a function to re-format them through a lot of trial and error. Every time I add a new agency to the mix, this function breaks and I have to modify it slightly. I plan to create a stable function sometime in the future. It takes two arguments: a directory of 'raw' files, and the name of an agency. It creates a tibble (spreadsheet) with two columns: `new_name` and `old_name`. It uses regular expressions to extract and format the agency name and record numbers according to the naming convention. Sometimes the files come in multiple parts so it identifies these and adds a part number to the end of the file name.

```R
cleanr <-  \(directory, agency){
  
  files <-  list.files(directory, pattern = "\\..+$") |> 
    tibble() |> 
    rename(old_name = 1) |> 
    #using regular expressions, re-format the record numbers
    mutate(new_name = str_replace(old_name, "A2", "A-2"),
           new_name = if_else(str_detect(new_name,"A-\\d{2}-"), str_replace(new_name, "A-", "A-20"), new_name),
           new_name = if_else(str_detect(new_name, "A-\\d{9}"), str_extract(new_name, "A-\\d{9}") |>
                                           str_replace("(A-\\d{4})(\\d{5})", "\\1-\\2"), new_name),
           new_name = str_extract(new_name, "(A-\\d{4}-\\d{2,5})"), # extract the formatted record number
           new_name = paste(agency, new_name, sep = "_"), #insert agency in front of record number
           new_name = paste0(new_name, str_to_lower(str_extract(old_name, "\\..{3,4}$"))), 
           new_name = paste0(directory, new_name),
           old_name = paste0(directory, old_name), #paste the directory in front
    ) |> 
    group_by(new_name) |> 
    mutate(dup_count = n(), #count duplicated record numbers (parts)
           dup_num = row_number()) |> #count the number of parts
    ungroup() |> 
    #label parted records
    mutate(new_name = ifelse(dup_count > 1, paste0(str_remove(new_name, "\\..+$"), "-", dup_num, str_extract(new_name, "\\..+$")), new_name)) |> 
    select(old_name, new_name)
  
  return(files)
}
```

This function makes the workflow for renaming files really straight forward. First, we run the re-naming function. We can then verify that the names look as we expect them to. This is really important as once we have renamed the files, we can't go back! Once satisfied the file.rename and file.copy functions from base R (you guessed it!) rename and then copy the files to a general folder storing ATIPS with cleaned names by agency. I choose to keep the rename and copy separate from the cleaning function so I can verify everything is working as expected at each step. This should leave us with a folder of files named with unique identifiers.

```R
clean <- cleanr("data/atip-files-raw/IRCC-raw-Aug25/", "IRCC") #run the cleaning function

file.rename(from = clean$old_name, to = clean$new_name) #rename the files

file.copy(from = clean$new_name, to = str_replace_all(clean$new_name, "atip-files-raw/IRCC-raw-Aug25/", "atip-files/IRCC ATIPS/"), overwrite = FALSE) #copy the files to general folders

```
## Link with metadata
The next step is to link these files with available metadata. This maximizes the utility of ATI/FOI documents in so many ways! They become easier to cite, easier to find, and easier to contextualize using request summaries. Again, I created a utility function to link files with their metadata.

This function generates metadata for files located in a specified directory. The function extracts the unique identifiers as well as the metadata contained in them: the agency and the request number. It also creates a separate column for the local path to each file.

This metadata is then combined with a dataset of summaries fetched from a [mirror of historical request summaries](https://github.com/lchski/gc-ati-summaries-data) created by [Lucas Cherkewski](https://lucascherkewski.com/). Before it can be merged, the mirrored data must be cleaned to deal with various formatting inconsistencies and oddities in the original source. The request numbers and agency names need to line up between the locally generated metadata and request summaries because we use them to join the two.

Finally, we ensure that our metadata is formatted for compatibility with the IA. Some metadata fields are required by IA and others are optional, but they need to be formatted in a particular way. The function outputs a spreadsheet with the following fields: `identifier`, `file`, `description`, `title`, `creator`, `date`, `adaptive_ocr`, `better_pdf`, `mediatype`. You can read about all the different metadata fields on the[ IA developer website](https://archive.org/developers/metadata-schema/). To successfully upload, our spreadsheet must include a unique identifier, local paths to the files we want to upload, and ideally a media type. The first two are required and the media type cannot be changed once a file has been uploaded. For me, it is easiest to just upload all the metadata at the same time as the file.

```R
metamakr <- function(path){

#list the files that are in the specified directory
files <- list.files(path) %>% 
  tibble(file = .) %>% 
  mutate(internal_id = sprintf("%03d", seq_along(1:nrow(.))))

#parse the file name, separating the agency and request number
parse <- files %>% 
  separate(file, c('agency', 'request_number'), sep = "_") %>% 
  mutate(request_number = str_remove(request_number, "\\..+$"), 
         request_number = str_extract(request_number, "A-\\d{4}-\\d{1,5}"),
         request_number = str_remove(request_number, "(?<=A-\\d{4}-)0+")) ## removes extra zeros
         

#re-join the file names and the parsed components
#specify internal id as file name without the file extension
join <-  full_join(files, parse, by = 'internal_id') %>% 
  relocate(internal_id, .before = file)%>% 
  mutate(identifier = str_squish(str_remove(file, "\\..+$")),
         file = paste0(path, file))

#read in summaries
summaries <- summaries <-  read_csv("https://raw.githubusercontent.com/lchski/gc-ati-summaries-data/main/ati-summaries.csv") %>%
  mutate(agency = if_else(str_detect(owner_org, "-"), str_to_upper(str_extract(owner_org, ".+(?=-)")), str_to_upper(owner_org)),
         agency = if_else(agency == "JUS", "DOJ", agency), #rename justice
         agency = if_else(agency == "CIC", "IRCC", agency), #rename immigration
         request_number = case_when(
           str_detect(request_number, "A\\d{2}") ~ paste0("A-20", str_extract(request_number, "(?<=A)\\d{2}-\\d+")),
           TRUE ~ request_number),
         request_number = if_else(str_detect(request_number, "\\/"), str_extract(request_number, "^.+(?=\\/)"), request_number), #deal with doubled request numbers separated by a slash
         request_number = str_remove(request_number, "(?<=A-\\d{4}-)0+"),
         request_number = str_extract(request_number, "A-\\d{4}-\\d+"))


#join files in directory with the metadata from summaries as well as custom metadata fields for the internet archive
metadata <- left_join(join, summaries, by = c('request_number', 'agency')) %>%
#format the request number and date fields to add leading zeros
  mutate(request_number = paste0(str_extract(request_number, "A-\\d{4}"), "-",
sprintf("%05d", as.numeric(str_extract(request_number, "(?<!A-\\d{4})\\d+$")))),
         date = paste(year, sprintf("%02d", month), sep = "-"),
         #add english and french descriptions
         description = paste(summary_en, summary_fr, sep = "\n\n"),
         adaptive_ocr = "true", #extra IA metadata fields
         better_pdf = "true",
         mediatype = case_when( #assign mediatype according to file extension
           str_detect(file, "\\.pdf$") ~ "text",
           str_detect(file, "\\.xls$") ~ "data",
           str_detect(file, "\\.xlsx$") ~ "data",
           str_detect(file, "\\.kml") ~ "data",
           str_detect(file, "\\.wav$") ~ "audio",
           str_detect(file, "\\.mp3$") ~ "audio",
           str_detect(file, "\\.cda$") ~ "audio",
           str_detect(file, "\\.mov$") ~ "movies",
           str_detect(file, "\\.wmv$") ~ "movies",
           str_detect(file, "\\.avi$") ~ "movies",
           str_detect(file, "\\.mp4") ~ "movies",
           TRUE ~ "unknown"),
         title = str_remove(identifier, "\\w+_"), #make the request number the title
         creator = owner_org_title) %>% #make the department the author
  select(identifier, file, description, title, creator, date, adaptive_ocr, better_pdf, mediatype)

}
```

The utility function does most of the heavy lifting so creating the metadata is pretty simple. All we need to do is provide the function with a path to the folder storing our freshly re-named files. All files waiting for upload are then saved to a .csv in an "upload queue" folder.

```R
uploaded <- read_csv("data/uploads/ia-uploaded.csv") |> 
  mutate(across(where(is.logical), ~tolower(as.character(.x))))

### Replace with path to relevant folder
path <- "data/atip-files/IRCC ATIPS/"

meta <-  metamakr(path) #run the utility function

meta |> #write a .csv to the upload queue
  write_csv(paste0('data/uploads/upload-queue/IRCC-', Sys.Date(), '.csv'))
```
## Upload to Internet Archive
Now it's time to upload. IA has both a python library and a command-line interface (CLI) tool that support bulk uploads. You will need an Archive.org account to use these tools. 

The whole process can easily be done in Python. I'm more comfortable in R so I use the `system()` function to conveniently serve commands to the CLI tool. To download the CLI tool, go to the Terminal tab in RStudio (in the same pane as your Console) and run the following two lines:
- `% brew install internetarchive` – this requires the Homebrew package manager. If that means nothing to you just replace 'brew' with 'pip' and it should work.
- `% ia configure` – it will ask you to log in with your IA credentials.

  Note: I had the CLI tool installed already but needed to re-install it via R to make this process work.
  {: .prompt-info}

I maintain a .csv file with a record of all the documents I've previously uploaded  to ensure that I'm not duplicating efforts (and straining IA's resources). The `anti_join()` simply removes rows that have previously been uploaded. If you haven't uploaded anything yet, you can skip this step initially but this file will be created when we upload.

The most straightforward way to upload in bulk to the internet archive is to go to your terminal (I'm on a Mac) and run the command `% ia upload --spreadsheet=file.csv`. I switched to the for-loop written out below because IA's backend often gets overloaded and fails to upload files. When this happens there is no automatic way to identify which of the files successfully uploaded or not. You just have to re-upload the whole spreadsheet, which of course deepens the problem of overloading IA's infrastructure. 

  If you are reading this and happen to have a lot of money, consider donating to the Internet Archive!
  {: .prompt-tip}

Our for loop addresses this problem by running each file independently. It iterates across each row in our 'upload' dataframe, writes that single row to a temporary .csv, and runs the `% ia upload` command. It uses the `--retries` flag to give each file 20 chances to successfully upload before moving on to the next file. Including`2>&1` in the command redirects error messages to the same 'stream' as the other outputs so we can capture them in one place. If the output includes "%100" we know that our upload was successful so we can add it to our log of successful uploads by appending it to `ia-uploaded.csv`, if not we have an error. This strategy allows us to keep track of successful uploads so if we run into any errors (almost guaranteed if you're uploading a lot of documents), we can just run the whole code chunk again and the anti-join will automatically remove files that were successfully uploaded from the queue. In the future, I'll wrap all of this in a while-loop so it just keeps iterating until all the files are uploaded.

```R
path <- "data/uploads/upload-queue/IRCC-2023-08-25.csv" #path to upload queue

upload <- path |> #read in the path
  read_csv() |> 
  mutate(across(where(is.logical), ~tolower(as.character(.x)))) #convert logical columns to lowercase text

done <- read_csv("data/uploads/ia-uploaded.csv") |> #read in the list of uploaded files
  mutate(across(where(is.logical), ~tolower(as.character(.x)))) #convert logical columns to lowercase text

upload <- upload |> #remove files that are alredy uploaded from the queue
  anti_join(done)

for(i in 1:length(upload$identifier)){ #for each row

  tmp <- upload[i, ] |> #write a temp file
    write_csv('data/cache/tmp.csv')
  #upload the temp file
  command <- "ia upload --spreadsheet=data/cache/tmp.csv --retries 20 2>&1"
  
  result <- system(command, intern = TRUE)
    
  if(str_detect(result[2], "100\\%")){ #if successful add the file to the uploaded list
    write_csv(tmp, paste0('data/uploads/ia-uploaded.csv'), append = TRUE)
    print(paste("Successfully uploaded", upload$identifier[i]))
  } else { #if unsuccessful indicate that in the console
    print(paste("Error at", upload$identifier[i]))
  }
}
```

## Optional: Upload to Google Pinpoint
The internet archive is a fantastic way to store lots of documents in a (hopefully) permanent way. While the IA does apply optical character recognition (OCR) to extract the text of uploaded .pdf files, there isn't a convenient way that I've found to search for content across a collection of files. This is where Google Pinpoint comes in. Unfortunately, the full version of this tool is only available to journalists and academics at the moment but I have found it incredibly useful in my research. I teamed up with my colleagues Kevin Walby and Alex Luscombe to publish a [trove of Canadian ATIP files](https://journaliststudio.google.com/pinpoint/search?collection=272a84cac9d13321) in .pdf format that anyone can access and explore. Not only does Pinpoint offer really advanced OCR, but it leverages Google's fancy AI to improve search functionality and identify people, places, and organizations mentioned in the documents you upload (with good but not perfect accuracy).
# Conclusion
I've put a lot of effort into figuring out how to archive ATI/FOI documents. While my strategy certainly doesn't fix existing problems with Canada's ATI system, I'm glad I did it. It saves me a lot of time in the end. Having these documents in one place and a searchable format helps find relevant information faster and cite those documents easily when they're used in published work. It is also simple to share the documents in my library with journalists, other academics and anyone else that is interested. If you find this useful and choose to archive your own documents, let me know. I would love to hear from you.