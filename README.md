# Add Tasks by Text through Gmail's API

I manage my tasks from my desktop, but it would be nice to add new tasks on my phone, out in the wild, as I think of them.

### Taskserver
I tried getting through the taskserver tutorials, I really did, but it ended up taking too much effort to try and figure it all out. I don't even need a fully synced server setup anyway, I just want to be able to add a few tasks from my phone every now and then.

I decided to try MacGyvering a solution for myself.

## Text
I recently found out one can send a text to an email address. I'm already in the habit of writing notes to myself by texting my own number. It wouldn't be much of a mental shift to start texting tasks to myself in a similar fashion.

### Contact
I'll be sending these texts to a subaddressed version of my own email address. I've created a new contact in my phone, named Task Add, and gave it an email address of 

    myemail+taskadd@email.com

All of my new tasks will be sent to this contact.

### Format
* Every text consists of one or more lines of text.
* The first line of text describes the task to be added.
* a few Modification types can be placed anywhere on the first line without disrupting the description.
  * Tags start with "+"
  * Project starts with "pro:"
  * Priority starts with "pri:"
* Annotations will be added for every additional line of text.

#### Examples:
```
testing tags +tag +youreit
```
```
complex item pri:H pro:important.project +doitnow  
it has an annotation
```
```
mixing a few +chaotic things together proj:testing to see if it works pri:m
```

## Gmail 
### Filters & Labels
I'm using Gmail's filters to mark each task with two labels: a permanent "**taskadd**", and a temporary "**tasknotread**".
Two filters need to be created, because gmail is only capable of adding a single label per filter.
```
From: myphonenumber@sms.provider.net
To: myemail+addtask@email.com
AddLabel "**taskadd**"
```
```
From: myphonenumber@sms.provider.net
To: myemail+addtask@email.com
AddLabel "**tasknotread**"
```
> Pro Tip: Send a test text from your phone number if you need to see its email address.

### API
There are a few things that need to be done before using the Gmail API with python.

* [Create a Google Cloud Platform project.](https://developers.google.com/workspace/guides/create-project)
* [Enable the Gmail API.](https://developers.google.com/workspace/guides/enable-apis)
* [Create OAuth client ID credentials.](https://developers.google.com/workspace/guides/create-credentials#oauth-client-id)
* [Install Google cilent library](https://developers.google.com/gmail/api/quickstart/python#step_1_install_the_google_client_library)

I saved my credentials right in the hooks folder. I wasn't sure where I should be saving it, so why not make it convenient for my code to access.

## Taskwarrior
I created a new hook file in **.task/hooks**, ensured its name starts with "**on-launch**", and made it executable.

    touch ~/.task/hooks/on-launch.gmailtaskadd
    chmod +x ~/.task/hooks/on-launch.gmailtaskadd

> Read more here: [Hook Author's Guide](https://taskwarrior.org/docs/hooks_guide/)

### General Code Flow
* find reasons not to run code
  * code is already running
  * code ran too recently
  * no internet
* connect with gmail resource
  * get credentials authorized with sign-in
  * store authorization in token for later
* get messages labeled **taskadd** and **tasknotread**
  * our initial query gives us a list of message ids
  * we use these ids to get the full message data
* parse messages into tasks
  * tokenize first line of message
  * filter out modification tokens
  * the remaining tokens are the description
  * any additional lines are annotations
* bash tasks into taskwarrior
  * run each part of the message through shlex.quote()
  * combine the sanitized parts into a bash script
  * run bash script through a subprocess shell
* clean up inbox and settings
  * remove **tasknotread** label
  * remove message from inbox
  * reset flags and timers

> Reminder: You can back up your task library by making a copy of the .task directory!

Goto line 50 and update the filepath to your own credentials file.
