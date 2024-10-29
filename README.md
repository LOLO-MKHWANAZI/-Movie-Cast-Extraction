# Movie-Cast-Extraction

# Movie Cast Extraction Script

This script is designed to scrape and extract information about cast members from a movie’s IMDb page, retrieving specific details such as the actor's name, birth date, birth location, bio, and job categories. It processes data for up to 67 cast members and outputs the information in a structured format.

## Key Steps

1. **Define the Movie Cast URL**  
   Specify the IMDb URL of the movie’s cast page. This URL will serve as the main source for scraping.

2. **Extract Cast Members’ Bio URLs**  
   The script first scrapes the cast page to get individual URLs for each cast member’s bio page by selecting the links associated with each profile image.

3. **Select Top 67 Members**  
   To keep the data manageable, the script selects the first 67 cast members from the scraped list.

4. **Initialize Results Storage**  
   Create an empty list to hold the extracted information for each cast member.

5. **Loop Through Each Cast Member URL**  
   For each URL:
   - **Fetch the HTML Content**: Attempts to read the bio page of the cast member. If there’s an error, it skips to the next member.
   - **Scrape Data Fields**:
     - **Name**: Extracted from a `<span>` element inside an `<h1>` header.
     - **Job Categories**: Scraped from list items within a job categories `<ul>`, displaying each job separated by " | ".
     - **Bio**: Retrieved from a `<div>` containing the cast member’s biography content.
     - **Born Date**: Date of birth is captured from a `<div>` specified with a birthdate data attribute.
     - **Born Location**: Extracts the location within the birthdate `<div>`.
   - **Store Data in a Tibble**: Each cast member’s details are stored as a row in a tibble (table) for structured output.

6. **Save the Results**  
   The tibble containing all the cast member information is saved as a CSV file, providing an organized, structured format for analysis or further processing.

## CSS Selectors Used

- **Name**: `'h1[data-testid="hero__pageTitle"] .hero__primary-text'`
- **Job Categories**: `'.ipc-inline-list.sc-ec65ba05-2 li'`
- **Bio**: `'.ipc-overflowText[data-testid="bio-content"]'`
- **Born Date**: `'div[data-testid="birth-and-death-birthdate"]'`
- **Born Location**: `'div[data-testid="birth-and-death-birthdate"] a'`

## Requirements

- **Libraries**: `rvest`, `dplyr`, `readr`, `tibble`
- **Error Handling**: The script uses `tryCatch` to handle errors in HTML fetching gracefully, allowing it to skip any inaccessible URLs.
- **Output Format**: The final output is a CSV file containing columns for URL, Name, Born Date, Born Location, Bio, and Job Categories.

## Example Usage

To run this script, simply execute it in an R environment with the required libraries installed. Adjust the `movie_cast_url` to the IMDb cast page URL of the movie you wish to analyze.

---

This script provides a streamlined and effective way to gather detailed information about movie cast members for data analysis or cataloging purposes. It handles data extraction and formatting automatically, making it easy to extend for various IMDb pages.



```R
# Load necessary libraries
library(rvest)
library(dplyr)
library(readr)

# Define the URL for the cast page of a movie
movie_cast_url <- "https://www.imdb.com/title/tt0362270/fullcredits?ref_=tt_cl_sm"

# Get the URLs for each cast member's bio
bio_urls <- read_html(movie_cast_url) %>%
  html_elements('.cast_list .primary_photo a') %>%
  html_attr('href') %>%
  paste0('https://www.imdb.com', .)

# Take the first 67 cast members for demo purposes
top67_bio_urls <- head(bio_urls, 67)

# Initialize an empty list to store results
results <- list()

# Loop over each cast member's bio URL
for (bio_url in top67_bio_urls) {
  message("Scraping URL: ", bio_url)
  
  # Read the HTML for the bio page
  bio_html <- tryCatch(read_html(bio_url), error = function(e) NULL)
  if (is.null(bio_html)) next  # Skip if there's an error loading the page
  
  # Parse the bio page to extract details
  name <- bio_html %>%
    html_element('h1[data-testid="hero__pageTitle"] .hero__primary-text') %>%
    html_text2() %>%
    replace_na("NA")
  
  job_categories <- bio_html %>%
    html_elements('ul.ipc-inline-list--show-dividers.sc-ec65ba05-2.joVhBE.baseAlt .ipc-inline-list__item') %>%
    html_text() %>%
    paste(collapse = " | ") %>%
    replace_na("NA")
  
  bio <- bio_html %>%
    html_element('.ipc-overflowText--pageSection[data-testid="bio-content"]') %>%
    html_text() %>%
    replace_na("NA")
  
  born_date <- bio_html %>%
    html_element('div[data-testid="birth-and-death-birthdate"]') %>%
    html_text() %>%
    replace_na("NA")
  
  born_location <- bio_html %>%
    html_element('div[data-testid="birth-and-death-birthdate"] a') %>%
    html_text() %>%
    replace_na("NA")
  
  # Save results in a tibble
  bio_tibble <- tibble(
    url = bio_url,
    name = name,
    born_date = born_date,
    born_location = born_location,
    bio = bio,
    job_categories = job_categories
  )
  results[[bio_url]] <- bio_tibble
}

# Combine all tibbles into a single data frame and save as a CSV
results_df <- bind_rows(results)
write_csv(results_df, "cast_member_details.csv")
```

![MOVIE CAST](https://github.com/user-attachments/assets/c55efbdd-eab1-4833-9b5a-5dcb0f140fed)
