## Background

In [my website theme](https://github.com/StevenTammen/spartan), I want to be able to support timestamp links on the page that change the time in a page-embedded YouTube video (rather than opening the video in a new tab, and so on). Well, I actually want to support both (and let users choose via a cookie-based setting), but since the latter is easy, it really is mostly a matter of figuring out the page-embedded links.

I had found [a simple video tutorial](https://www.youtube.com/watch?v=EVNTyuZrAlg) going over how to do this with a single embedded video, but I want to support multiple embedded videos on the same page. The [documentation for YouTube's Iframe API](https://developers.google.com/youtube/iframe_api_reference) is all also written mostly in terms of a single video. So going there wasn't super helpful in and of itself.

After some brief searching, I came across [this gist](https://gist.github.com/bajpangosh/d322c4d7823d8707e19d20bc71cd5a4f), which is much closer to what I wanted, and proved an excellent start.

Here is what I set out to do:

- Support page-internal video timestamps via calling the `seekTo()` function on various `Player` objects dynamically
- Set up event listeners so that when any `Player` is played (either manually, or via some timestamp link), all other playing videos are paused
- Support jumping to the currently playing video (by figuring out which video is playing in JavaScript, and then navigating to that video's element Id on the page, in the sense of [htts://www.somesite.com/some-page/#myvideoid](#)). By "jumping" I mean changing the scroll position on the page---triggering this JavaScript function will scroll so that the currently playing video is brought into view.

I never figured out how to get the code to work with already-existing iframes, although I'm guessing it is possible. So right now, I just have the `Player` constructors dynamically load the embedded videos into wrapper `<div>` elements, and this works fine. I do not think it matters much---it just changes how you have to set up your static site generator templates/shortcodes.

## This example

The HTML file in this repo (very creatively named `example.html`) demonstrates a few things. In no particular order:

- How to embed multiple YouTube videos on the same page using YouTube's Iframe API to load the iframes dynamically. Here, I give each container `<div>` that will house a dynamically-loaded iframe a class attribute of `embedded-youtube-video` and an id attribute matching each respective video Id. (Note that YouTube video Ids can contain hyphens, which is OK in CSS selectors). You need to specify these things on each container `<div>` for the JavaScript to work properly (since they are needed for a `querySelectorAll()` call, and so on).
  - I added a bunch of Lorem Ipsum text to ensure that the dynamic iframe loading does not cause page focus to jump around. I've had issues with this behavior in the past with other iframes I've embedded on pages (especially those without full lazy loading), so I wanted to check. Thankfully, no issues here.
- How to set up links to change the current player time. You can have different links change times for different videos on the same page without them interfering with each other.
  - These links call a helper function that takes in the YouTube video Id (a string) and a time (in seconds) and then makes the specified video play at that time. The function can be reusable like this due to a JavaScript map object (mapping the video Ids to their respective `Player` objects) that gets made once the YouTube Iframe API library finishes loading.
  - Using these links to play the video at a particular time works fine even if the video was not playing already, and even if the `Player` has never been played as of yet.
  - Using the links to play the video at a particular time does not adjust page focus (that is, instantly jump to the affected video). I added more Lorem Ipsum text (in between the top video and its timestamp link) to make sure this was so. 
    - There are sort of pros and cons to not immediately jumping to the affected video.
    - I prefer to not immediately put the focus back on the video so that I will be able to have a timestamp link under each content header on my pages, which will, when clicked, play the embedded video at the start time for that section. This lets people instantly get an audio version of whatever content they are looking at, without messing with their scroll position.
    - This will also be relevant for you if you ever put transcripts on your pages, fully hyperlinked to adjust the embedded video's play time. This is very useful, as it gives people a way to text-search your video content, and then immediately jump to the relevant video time for any matches. If you had the focus shift back to the video, it could mess with searching UX.
- The JavaScript you need to have in order to set up a link in your site's menu to take you to the currently playing video.
  - This is a large part of why I do not suggest automatically jumping to the playing video when you change the time via timestamp hyperlink: the vast majority of the opportunity cost in not doing such can be completely avoided simply by adding a link like this.
  - Then you can jump back to the video when you want to, and not when you don't want to. Best of both worlds sort of thing.
- What JavaScript you need to have in order to set up event listeners that enforce the rule "Only one YouTube video can be playing at a time." Basically, we implement this by making it so that playing a video (manually, or via timestamp link) will automatically pause any other video that is playing.

## Limitations

This is a pure HTML/JavaScript example, which will be maximally portable across different static site generators (Hugo, Gatsby, Jekyll, etc.) and content management systems (WordPress, Blogger, etc.)

Hand-coding the `<div>`'s to embed videos page after page doesn't make very much sense, in my opinion, but this structure will integrate easily into other automation workflows, and that is how I recommend people utilize the concept. I will be integrating a version of this behavior into my own workflow (which has multiple components: [my Hugo theme](https://github.com/StevenTammen/spartan), my [Hugo preprocessor Python application](https://github.com/StevenTammen/hugo-preprocessor), and my [command line video processing Python application](https://github.com/StevenTammen/command-line-video-and-audio)).

So, for example, when I am done writing the code to automate my particular workflow, I will be able to run a single command to process a video recording and upload it to YouTube, and then save the returned video Id back to the content file, embed the YouTube video based on that video Id, and automatically calculate and set up timestamp hyperlinks for all the content headers gone over in the recording. Then, after waiting a while for YouTube to process the video and automatically generate a text transcript, I can run another command to download the automatically-generated video transcript and make all the timestamps therein hyperlinks too.

So basically, everything you need to do to utilize this concept can be completely automated (and probably should be completely automated). But even so, the video timestamp hyperlinks in all that will still basically function like those in this simple example.
