### Too curious cats
>Cats are curious by nature and this cat was more or less curious on his travels. He came back home but he brought something with him. Not ticks but something is hidden in this image. Can you find it? http://sec-mooc-1.cs.helsinki.fi/cat_3/fd8101ef-bfce-4a68-8895-7d8406dc0941/cat03.bmp


Another stego challenge but this time it's a `.bmp` and not `.jpg`. Honestly this challenge for me was just blindly googling for and trying out different stego programs. I actually found the program (`zsteg`) that ended up solving this quite quickly but it wouldn't install properly so I skipped it. After exhausting all other options I came back to it and managed to install it on an old vm running on another computer.. All in all a very frustrating challenge as I don't know much about steganography.

```zsteg -a cat03.bmp```

Which included this:

```b1,bgr,lsb,xy       .. text: "$lsbsecretrandom4027387520957partflag"```

The above string without the '$' was the flag.

I was actually only 1 click away from solving this with `Stegsolve` about an hour or so before I finally did manage to solve it. When looking at the image's grey bits in Stegsolve you can clearly see the top row has something in it. Now looking at each of the color layers you can see that same row is different in layer 0 compared to other layers. However the mistake I made was stupidly not checking for different bit plane orders. The correct one ended up being `BGR`.


