# Extract your Mac iMessage database for easy data analysis

## Purpose
Welcome to his repo. Over the past years I've been writing and editing the code to access, extract and transform the data in your Mac's iMessage database in order to create a dataframe that you can use to perform all sorts of fun data analysis.

## Version
3.1

## Getting Started
You need a Mac for this process.

You need familiarity with Python and [jupyter notebooks](https://jupyter.org/try-jupyter/notebooks/?path=notebooks/Intro.ipynb) to use this code.
In the future I might package the whole ETL notebook into a script you can run, but that's not a big priority right now.

1. Fork this repo
2. Clone to your local machine
3. Open jupyter notebook, load and run the notebooks.

The repo has two notebooks. One is the ETL notebook that you need to run once in order to extract, transform and save a clean, data-analysis-friendly dataframe. The other is a notebook with some analysis ideas and code. 

## Reading
You can read the first blog post I wrote that explains that fundamentals of the code [here](https://medium.com/@yaskalidis/heres-how-you-can-access-your-entire-imessage-history-on-your-mac-f8878276c6e9). Even though the code itself has since been updated, the base code and process is the same.

You can also read about some fun analysis you can do once you have the clean dataframe [here](https://medium.com/@yaskalidis/fun-things-you-can-learn-about-yourself-and-from-your-messages-5101631a8e20)


## How Apple stores your messages database
The main data entities we're working with here is the messages, the chats and the contact information.

```messages```: table that has information about each unique message sent or received. The information includes the text (see below for more on this), time and whether the message was sent or received.

```chat_message_joins```: table that maps each unique ```message_id``` to a unique ```chat_id```.

```chat_handles_join```: tables that maps each unique ```chat_id``` to the list of ```handle_id```s that participate in that chat.

```handles```:table that maps each ```handle_id``` to the contact information - phone number or email.

## High level overview of the extraction and transformation process

In the ```messages``` table, each unique message has a ```ROWID``` which I rename ```message_id``` since it's a unique identifier for messages. 
I join the ```messages``` table to a table called ```chat_message_joins``` using the ```message_id```. This join gives me the ```chat_id```, representing the unique chat that each message was sent.
Finally, the remaining information that is missing is the participants in each chat, and hence the people that received or sent each message.

For that I use two tables: the ```handles``` table that has a unique ```handle_id``` for each unique handle that you have interacted with and that handle's contact information. 

The ```chat_handles_join``` table bridges the handles information back to the messages because it includes which handles are participating in each ```chat_id```.

So the discovery process can look something like this:
get the ```message_id``` from the ```messages``` table, get the ```chat_id``` of that message from the ```chat_message_joins``` table, get the handle_ids of the participants in that chat from the ```chat_handles_join```, and finally translate the handle_ids to contact information (phone numbers and email you can read and recognize) using the ```handles``` table.

## Notes

* Some messages in the ```messages``` table have a ```NULL``` text field even though the message had text when I sent or received it. This seems to be something that was duplicated, since the repo [here](https://github.com/niftycode/imessage_reader) also has a similar issue. 
I solve for this in this new version of the code by discovering and extracting the message body field inside a column called ```attributedBody```. It's totally perfect but it works pretty great. To distinguish, I create a new column called ```inferred_text```.

* The recipient field is also generrally ```NULL``` when the message is a sent one (vs received). After some brainstorming and few tries, I filled that information by looking at the participants in the chat. If it is a private chat (i.e., you and one more person) then if the message is sent, the recipient is the other person in the chat. If the chat is a _group_ chat I decided to have the recipient simply as 'group-chat', since there is no single recipient (which mnight be why Apple decided to ```NULL``` that field). In the future I do want to iterate on this since there is a lot of information lost by simply calling it 'group-chat': I will explore having the column be a list that sometimes has one recipient and sometimes more. 

* If someone you know messages you sometimes from their phone number and other times from their email (e.g., Apple ID) then Apple assigns two different ```handle_id```s. That's why I added Step 5 in the ETL notebook where you can manually create a dictionary that maps various contact information to the person's name. That way, when you want to do analysis on your messages with a specific person you query for the person's name, and the code will include all messages sent and received from both the phone number and the email.
* To make things slightly more complicated, Apple will store a differnet handle_id even if the same phone number messages you once on iMessage (blue buble) and once on SMS (green bubbles). Even when sending an iMessage, sometime the messages arrives as an SMS due to e.g, poor connection issues. So you have to be careful with that too. Again, Step 5 in the ETL notebook should take care of that, and when you query for a person's name you will get both SMS and iMessages.

* Note that some of the messages in the final dataframe will be things other than messages sent between people. This includes automated messages you might receive from services (like 2-fac authentication codes, restaurant reservation confirmation messages etc) as well as even more exotic thigns such as notifications that someone has started or stopped sharing their location with you or Apple Watch competitions. 

## Updates from Previous Version

Updates in 3.1
* I included a column that detects whether the "message" was a [reaction](https://support.apple.com/guide/messages/use-tapbacks-icht504f698a/mac) and, if so, what type of a reaction.
This works for reactions sent from English language iPhones, and doesn't work for Greek (yet). This detection relies on the text of the message.

Updates in 3
* Many more messages now contain text. This can enable you do to sentiment or other text analysis. My best impression is that Apple changed the way that it stores the message text in the database at some point, and moved towards runtime encoding. After lots of trial and error, I was able to extract a pretty good approximation of the English text from another column in the table. The big limitation right now is that this works only for English characters. I tried it with Greek and wasn't successful. 
* The sent messages now have their recipient in the data. If you sent the message in a group chat, it is simply denoted as 'group-chat' (for now, see below for future work)


## Future Work
* When the message is sent in a group chat, right now I simply have the recipient as 'group-chat'. I want to edit that and perhaps include the list of recipients. This way if you want to calculcate the numebr of messages between you and another person in both private chats as well as group chats you will be able to do so. 
* Right now, Step 3 of the ETL notebook is very slow because the code is not optimized. I want to make that more efficient.
* The messages in the final dataframe will include things like reactions and thread replies. For reactions, I have a sense of how I can detect them. I might include an indicator whether the message is a reaction, a thread reply or even things like photos in future editions. 
    *  **Update in 3.1**: Reactions are now detected and included in the ETL notebook.


<br>
<br>

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
## License

This project is licensed under the [Creative Commons Attribution-NonCommercial 4.0 International License](https://creativecommons.org/licenses/by-nc/4.0/).
