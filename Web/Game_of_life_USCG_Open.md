# Game of Life

## Description
> Having fun hacking and want a break? Why not play a little game?
> The Game of Life is a fun diversion you can use to simulate all sorts of interesting cellular structures.
> Maybe you'll find something else...

## Solution
Going to the Game of Life webpage, we see this.

![image](https://github.com/GageSch/CTFwriteups/assets/61923833/d38d00b7-0f8d-4c62-a2b9-6b76c5dfe1b7)

In the network tab, I notice that marking a square, hitting the play button, and hitting the reset button do not send any network traffic.
My gut instinct when first looking at this is that the solution is loading and saving the board. 
If I can upload a file, maybe there is some way I can get remote code execution.



I click the button to save the board while intercepting the request in Burpsuite. I see the server sends a file back with the extension *.golsave*

![image](https://github.com/GageSch/CTFwriteups/assets/61923833/f490964c-9eab-4b88-a5fc-85f7f432e6d3)

I use the *file* command to let me know what the actual file type is.

```command
┌──(gage㉿kali)-[~/Downloads]
└─$ file 64bb3c52d263d.golsave 
64bb3c52d263d.golsave: zlib compressed data
```

Knowing that the file is zlib compressed, I use cyberchef to get a quick look at the contents.

![image](https://github.com/GageSch/CTFwriteups/assets/61923833/a006b62b-88c1-416a-acd8-ab87b0c8ad01)

The output lets me know that this save file is a serialized PHP object. The second half of the object is used to encode the board game in an array, but the first half is supicious.
The comment in the second string lets us know that the developer intends to put a PHP callback hook there and the first string gives us confirmation that it is a hook.

PHP hooks are versatile and can do just about anything including reading/writing files and even running system commands.
Let's see if I can add my own hook and get remote-code execution.

![image](https://github.com/GageSch/CTFwriteups/assets/61923833/73c6afaf-6fd1-4786-a088-6f7da51ca7a7)

This first bit of PHP code calls *scandir()* which lists the contents of a directory, I have it pointing at the root directory right now.
I remember to include the semicolon and set the length to the right number before exporting it to a zlib compressed file.
Sending the file to the server, I get back confirmation our PHP code has been executed. 

![image](https://github.com/GageSch/CTFwriteups/assets/61923833/3cddb793-8c59-48fa-994b-bc4a670509b2)

In the output, we see a file called flag.txt - *how suspicious ;)*

I can use another built-in function to list the contents of this file, but I will make it easier for myself and use the *system()* command.

![image](https://github.com/GageSch/CTFwriteups/assets/61923833/183b6ce1-3401-4055-9a15-f5bea6222030)

This code prints out the contents of flag.txt and if all goes well we should see the output at the top of the webpage.

<img width="1681" alt="image" src="https://github.com/GageSch/CTFwriteups/assets/61923833/b8f9774b-2924-4668-b454-1e22e73cda1e">

