# Data Cleaning Project: Enron Emails

For this project I cleaned emails from five mailboxes (allen-p, arnold-j, arora-h, badeer-r, bailey-s) of the Enron Email Dataset from Carnegie Mellon University (https://www.cs.cmu.edu/~enron/). The raw data is 9,941 plain-text files, one email per file with no table structure. I used R to turn them into two tables and made a plot of the cleaned data. This README is my cleaning log: it explains each step and why I made each decision.

## Files:

- FirstFiveEmails.zip: the emails

- DataCleaningProject.qmd and DataCleaningProject.pdf: the cleaning script and its output

- emails_clean.csv: one row per email cleaned

- recipients_long.csv: one row per recipient linked to emails_clean.csv by id

## How to run it:

Unzip FirstFiveEmails.zip into the project folder so that allen-p, arnold-j and the other mailbox folders are next to DataCleaningProject.qmd then run the document. It uses base R only. Reading all the emails takes a minute.

## Why this data is messy:

- Unstructured: Each email is its own plain-text file. Headers come first, a blank line, then the message. There is no table.

- Multiple sources: The five mailboxes belong to five people and each stores its emails in its own folders for example _sent_mail, all_documents, discussion_threads and sent in allen-p.

- Names and formats: The same person can appear in different forms, such as "Arora, Harry" and "Harry Arora". Dates are text like "Tue, 4 Dec 2001 10:30:17 -0800 (PST)".

- Kinds of missing values: Some headers are missing from the email completely. Others are present but empty. After cleaning To is missing in 329 emails Cc in 9,023 and Subject in 1,894.

- Crammed columns: The To header packs recipients into one field. Splitting it turns 9,941 emails into 31,597 recipient rows.

## Explaining each part

The numbers match the parts of DataCleaningProject.qmd.

1. Find every email file. The script listed all the files in the five mailbox folders. It kept only the paths inside those folders, which gave 9,941 emails. The data was spread across thousands of files, not one table, and filtering by mailbox kept project files (the zip, the script, the output csvs) from being read as emails.

2. Read the header of one email. I wrote a function that read one email and pulled out From, To, Cc, Subject, Date, X-From and X-Folder. Each email was text and the headers ended at the first blank line. Long headers such as To wrapped onto lines that started with a space or tab, so I joined those lines together. Without that, only the first line of a recipient list would have been captured. If a header wasn't in the email, the value was NA. The files were read as latin1 text, which avoided errors from characters in the files.

3. Build one table. The function ran on every file. The results were combined into one data frame. I added an id for each email and mailbox and folder columns taken from the file path. The person and folder an email came from existed in the path, so putting that information in columns kept it available. The id was used later to link tables.

4. Make missing values consistent and clean sender names. Blank values were converted to NA so a header that was absent and a header that was empty were treated the same. R treated a string as a real value, so counts and filters would have been wrong if the two kinds hadn't been merged. After this step, To was missing in 329 emails, Cc in 9,023, and Subject in 1,894. For sender names, I removed the part in angle brackets and any quotes, flipped "Last, First" to "First Last," trimmed spaces, and made everything lowercase. This made "Arora Harry" and "Harry Arora" match as one person. The cleaning was rule-based, so it didn't merge nicknames or misspellings.

5. Fix the dates. Dates were text like "Tue, 4 Dec 2001 10:30:17 -0800 (PST)". I removed the weekday and converted the rest to a real date. The format accepted days with and without a leading zero. I ignored the time zone and kept only the date because I analyzed by day and year, where a few hours didn't matter. Real dates could be grouped by year, while text couldn't.

6. One row per recipient. The To field crammed addresses into one text value, so I split it on commas and made a second table, recipients_long, with one row per recipient and the email's id. It had 31,597 rows (the 329 emails with no To kept one NA row each). With the field as one value, each combination of recipients was counted as a different value, so nobody's recipient total could be counted. A separate table was better than columns because the number of recipients differed for each email. Addresses were lowercased so the same address always matched.

7. Handle dates. Twenty-two emails had dates before 1999. I set them to NA. The earliest valid email in these mailboxes was from December 1999, so dates before 1999 didn't fit the rest of the data. I didn't try to correct them because the true dates couldn't be recovered from the data, and guessing would have added information.

8. Save. The emails were sorted by date with missing dates at the end and saved as emails_clean.csv. The recipient table was saved as recipients_long.csv. Two tables kept each row meaning one thing. They joined back together by id.

## The plot:

The final plot shows the number of emails per year. Most emails are from 2000 and 2001 with fewer from 1999 and 2002. I was able to make this plot after converting the date text into real dates and handling the 22 impossible values.

## Decisions I did not make:

- Duplicate emails were not removed. The same email can be stored in folders of one mailbox, such as _sent_mail, all_documents, discussion_threads and sent. I kept them because each copy is a file in the source and deciding which one is the original is a judgment call.

- Names were not matched across people. I only standardized formatting.

- Original columns were kept. The original To and X-From text is still in emails_clean.csv so the cleaned values can be checked against the original data.
