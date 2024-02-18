## Background

In [my website theme](https://github.com/StevenTammen/spartan), I want to be able to support timestamp links on the page that change the time in a page-embedded YouTube video (rather than opening the video in a new tab, and so on).

I had found [a simple video tutorial](https://www.youtube.com/watch?v=EVNTyuZrAlg) going over how to do this with a single embedded video, but I want to support multiple embedded videos on the same page. The [documentation for YouTube's Iframe API](https://developers.google.com/youtube/iframe_api_reference) is all also written mostly in terms of a single video. So going there wasn't super helpful in and of itself.

After some brief searching, I came across [this gist](https://gist.github.com/bajpangosh/d322c4d7823d8707e19d20bc71cd5a4f) which is closer to what I wanted. However, my use case is a bit different, since all I really care about is calling the `seekTo()` function on various `Player` objects dynamically; I don't care about setting up any other event listeners ore doing anything more complicated than that single thing.

I never figured out how to get the code to work with existing iframes, although I'm guessing it's possible. So right now, I just have the `Player` constructors dynamically load the embedded videos into wrapper `<div>` elements, and this works fine. I do not think it matters much.

## This example

The HTML file in this repo (creatively named `example.html`) demonstrates a few things. In no particular order:

- How to embed multiple YouTube videos on the same page using YouTube's Iframe API to load the iframes dynamically. Here, I give each container `<div>` that will house a dynamically-loaded iframe a class attribute of `embedded-youtube-video` and an id attribute matching each respective video Id. (Note that video Ids can contain hyphens, which is OK in CSS selectors). You need to specify these things on each container `<div>` for the JavaScript to work properly (for a `querySelectorAll()` call, and so on).
  - I added a bunch of Lorem Ipsum text to ensure that the dynamic iframe loading does not cause page focus to jump around. I've had issues with this behavior in the past with other iframes I've embedded on pages, so I wanted to check. Thankfully, no issues here.
- How to set up links to change the current player time. You can have different links change times for different videos on the same page without them interfering with each other.
  - These links call a helper function that takes in the YouTube video Id (a string) and a time (in seconds) and then makes the specified video play at that time. The function can be reusable like this due to a JavaScript map object (mapping the video Ids to their respective `Player` objects) that gets made once the YouTube Iframe API library finishes loading.
  - Using these links to play the video at a particular time works fine even if the video was not playing already, and even if the player has never been played as of yet.
  - Using the links to play the video at a particular time does not adjust page focus (that is, jump to the affected video). I added more Lorem Ipsum text (in between the top video and its timestamp link) to make sure this was so. 
    - There are sort of pros and cons to not jumping to the affected video.
    - I prefer to not put the focus back on the video so that I will be able to have a timestamp link under each content header on my pages, which will immediately play the embedded video at the start time for that section. This lets people immediately get an audio version of whatever content they are looking at, without messing with their scroll position.
    - This will also be relevant for you if you ever put transcripts on your pages, fully hyperlinked to adjust the embedded video's play time. This is very useful, as it gives people a way to text-search your video content, and then immediately jump to the relevant video time for any matches. If you had the focus shift back to the video, it could mess with searching UX.

## Limitations

This is a pure HTML/JavaScript example, which will be maximally portable across different static site generators (Hugo, etc.) and content management systems (WordPress, etc.)

Hand-coding this doesn't make lots of sense, in my opinion, but it will integrate easily into other automation workflows, and that is how I recommend people utilize the concept. I will be integrating a version of this behavior into my own workflow (which has multiple components: [my Hugo theme](https://github.com/StevenTammen/spartan), my [Hugo preprocessor Python application](https://github.com/StevenTammen/hugo-preprocessor), and my [command line video processing Python application](https://github.com/StevenTammen/command-line-video-and-audio)).

So, for example, when I am done writing the code to automate my particular workflow, I will be able to run a single command to process a video recording and upload it to YouTube. Then, after waiting a while for YouTube to process the video and automatically generate a text transcript, I can run another command to grab the video Id, write it back to the content file, embed the YouTube video based on that video Id, automatically calculate and set up timestamp hyperlinks for all the content headers gone over in the recording, and then finally download the automatically-generated video transcript and make all the timestamps therein hyperlinks too.

So basically, everything you need to do to utilize this concept can be completely automated (and probably should be completely automated). But even so, the video timestamp hyperlinks in all that will still basically function like those in this simple example.
