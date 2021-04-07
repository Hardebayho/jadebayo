---
title: "Libavformat Tutorial" # Title of the blog post.
date: 2021-04-05T20:10:23Z # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true
menu: main
featureImage: "/logos/logo.png"
thumbnail: "/logos/logo.png"
shareImage: "/logos/logo.png"
codeMaxLines: 10
codeLineNumbers: false
figurePositionShow: true
series: "ffmpeg-tutorials"
categories:
  - Technology
  - Multimedia
tags:
  - libavformat
  - ffmpeg
  - multiplexing
# comment: false # Disable comment if false.
---

## Libavformat Tutorial

In this post, I would love to walk you through the libavformat API contained in FFMpeg so that you can understand the inner workings of the libavformat library. I will also try to show you how to use it with the other ffmpeg libraries, notably **libavcodec**.

### Terminologies In Order
Before I continue, I feel we need to become familiar with some terminologies you might encounter in the libavformat library. These are simply names that identify some sort of operation or entity in the multimedia programming world.

#### Stream (or Track)
A stream (or track) is simply a sequence of multimedia data. By multimedia, I mean audio, video and/or subtitles. A sequence of audio is called an audio stream or audio track. A sequence of video images is called a video track (or stream). Same for subtitles.

#### Packets
A packet is compressed data. These simply are bytes of data that contain information you give to a decoder to give you usable audio or video frames. Packets themselves are not really useful to a user, and only make meanings to the encoders/decoders that produce them.

#### Frames
If a packet is compressed data, a frame is the opposite. A frame means uncompressed data. A frame is what you would normally consume in your program, if you're playing an audio or video. It's also just a sequence of bytes, but the information contained can actually make sense to the user. For example, a video frame is an image that you can draw on the screen.

#### Containers
A container is what it is, a container. It "contains" streams (or tracks, whatever you want to call it) of data. What is good about containers and why they are useful is that they can contain multiple streams of data. This means that we can store audio streams and video streams inside just one file, instead of keeping audio and video files separate. Examples of container formats you might've already seen are **mp4**, **mkv**, **webm**, **ogg**, **mp3**, etc... There's a ton of them, and as you'll learn, some of them can contain both audio and video, while some of them can only contain audio. Some of them also require that they contained audio and/or video be of a specific format. Container formats also offer some sort of compression of the data they contain, so they also provide memory efficiency.

#### Multiplexing (or Muxing)
Multiplexing (or muxing for short) is the process of taking a container format, and putting audio/video streams inside it. A program that does this is called a multiplexer (or muxer). Why is muxing important? Well, if you think about it for a minute, audio and video streams as a sequence of bytes are really useless unless there is a way to correctly interpret the data inside of them in a meaningful way. Sometimes, we want to know the duration of the media. We could read the entire file and estimate the duration, but that would be pretty stupid considering all we need is just the duration. Containers store these data for us, so we could just read their headers and not have to read the entire file to get info about the media.
Another use for multiplexing is timing and synchronization. With video, they usually carry information about the frame rate, which talks about how many frames of video we would display in a second. For example, a video with a frame rate of 25fps means that we would have to display 25 frames every second, no more no less, else the video will either become too fast or too slow. This means that we have to display our video frames in an interval of (1/25) seconds as in our example. Muxers help us store these information inside the container and even more others. They also help with synchronization of the audio and video streams. You can simply think of muxers as the ones that produce containers from whatever streams you add to them.

#### Demultiplexing (Demuxing)
If muxers take different streams and puts them into a container, demuxer (or demultiplexers) take containers and decompose them into their individual streams. Demuxers read data written by muxers, and parse the data so that they can take out the streams and give you data contained within. Dead simple.

#### PTS (Presentation Timestamp)
This is something that is kind of interesting, because it actually took me some time to realize exactly what this meant. In order to be able to properly display a video frame, we need to know *when* to display the frame. We could have a thousand frames we want to render to our screen while playing a video. How do we know what frame to render, and what time we're supposed to render the frame? Enters **PTS (or presentation timestamps)**. These values specify exactly when a video and/or audio frame should be displayed and is usually contained in a video and/or audio stream. When a demuxer is breaking streams apart and returning the individual packets to the decoder, the decoder will need the pts values to know what frame to spit out next and also in what order. Just know for now that if you see videos and/or audio where the synchronization is fucked up, its very likely they messed up with the PTS when encoding and/or muxing.

#### DTS (Decoding Timestamp)
I didn't want to add this here since it's not really portable (i.e, it's only specific to ffmpeg and *maybe* some other libraries), but dts can help you when the frames you're receiving don't contain pts values. DTS in this case is more or less a guess by the decoder as to when you should display that frame. In most cases, the values of pts and dts will be the same. If the pts is valid though, you can mostly always discard the dts and just use that instead. If the pts is not valid, you can then use the dts returned by the decoder.
**NOTE:** *I'm not sure about the validity of this dts thing, mainly because I don't really understand it that much. If you think I'm wrong about it and you'll like me to correct this article, please send me an email with explanations to hardebahyho@gmail.com so I can fix it. This will help anyone that finds this article and is learning about multimedia programming so that they don't assume knowledge that is invalid.*

### Now the fun stuff
Right. Now that all the terminologies are out of the way, let's start tearing into the libavformat library.

#### What is libavformat?
Libavformat is a couple of things. First of all, it contains a ton of stuff that lets you do muxing and demuxing. Buh that's not where it stops, it also encapsulates (by encapsulate, I mean provide) IO protocols that lets you read media from any sane and standard IO protocols in existence. This means that libavformat in itself will allow you to:
+ Create containers and put audio and/video streams inside of it (subtitles too, if the format supports it).
+ Read data from containers and split the individual streams
+ Read metadata from media files and other info such as size of a video, the bit rate, frame rate, sample rate of the audio, and a ton of other information available in the container.
+ Read data from virtually anywhere. You can even read from a `uint8_t*` if you want.
+ Make going to the moon and back a *piece of cake*.

Now, multimedia is a complicated beast, and having knowledge about libavformat will help you a great deal in understanding how everything comes together. Even if you'll never use it in a commercial product (and I doubt you won't, cause it's just too cool. Too cool for you to want to use something else, trust me.), you'll still need all the knowledge it has to offer.

#### Where art thou codes?
Okay, you're now impatient since I promised to explain libavformat and I've been ranting about nonsense instead. Let's start.
Needless to say, make sure you include the `avformat.h` header file before continuing. If you're using C++, you'll need to include your header like this:
```c++
extern "C" {
#include <libavformat/avformat.>
}
```
And also needless to say, make sure you've set up your dev environment to link to the ffmpeg libraries and set up your include paths.

##### Opening a media file
The first thing you'll need before doing anything libavformat-related is an AVFormatContext. This is your ticket to the avformat functions, so create one for yourself:
```c++
AVFormatContext* format_ctx{nullptr};
```
If you don't initialize it to null at the beginning, the pointer will point to garbage and when you try to use it in the next operation we're about to perform, you'll wish you never learned programming, cause if the program doesn't crash, you'll get the most garbage values in existence and the things inside the format context will be undefined (I used to think the word undefined in C and C++ was weird, now I understand). Moral lesson is, always initialize your pointers **AS SOON** as you create them to avoid your lifespan being reduced by half.

```c++
extern "C" {
#include <libavformat/avformat.h>
}

int main() {
    AVFormatContext* ctx{nullptr};
	// Format ctx, URL, Input format, options
    int ret = avformat_open_input(&ctx, "file:/url/to/anywhere", nullptr, nullptr);
	
	ret = avformat_find_stream_info(ctx, nullptr);

    if (ret < 0) {
        fprintf(stderr, "We're out of luck! Libavformat can't help us here\n");
        return -1;
    }

    // Write more code, our path is valid
	
	avformat_close_input(&ctx);
    return 0;
}
```
What do we have here? Well, obviously we include our libavformat header file, and inside the main function we open the input. In the `avformat_open_input` method, we need to pass four things:
+ The format context
+ The url (path) to our media file
+ Input format
+ Options

The first two is obvious. The third parameter, however is an `AVInputFormat` structure that contains information about the media file container. Creating, populating and passing this value to libavformat means you're telling it not to try to figure out what format this file actually contains, and that you KNOW the format and things inside of it. Generally, I set this to null, since libavformat can figure out the format of the media file itself. The last parameter is also set to nullptr. This is for when you want to pass some options to configure the demuxer. I've never had any use for it, so I always set it to nullptr.

The `avformat_find_stream_info` is important. If you don't call this function after opening the input, the format context might not have information you need about the streams. Bottomline: Call the method once you open an input file.

Lastly, we close the input so that ffmpeg can clean up whatever memory it's used.

##### Reading metadata
Okay, let's read metadata from a media file. In case you don't know what metadata means, metadata simply is some textual information that may or may not be useful to you. It was put there by whoever muxed the audio and/or video and typically contain information about the media. Information like who made the media, the software used, the artist (for music or audio content), and other kinds of info.

The first thing we need to do is to open the media file and read the stream info. We know how to do that from the last section, so we're going to learn how to read metadata in this one:

```c++
// Reading single metadata
AVDictionaryEntry* entry{nullptr};
if ((entry = av_dict_get(ctx->metadata, "artist", nullptr, AV_DICT_IGNORE_SUFFIX))) {
	printf("%s => %s\n", entry->key, entry->value);
}

// Reading all the metadata
AVDictionaryEntry* entry2{nullptr};
while ((entry2 = av_dict_get(ctx->metadata, "", entry2, AV_DICT_IGNORE_SUFFIX))) {
	printf("%s => %s\n", entry2->key, entry2->value);
}
```

Here we can see that it's relatively easy to extract metadata from a media using ffmpeg. All we need is a `AVDictionaryEntry` and the format context's metadata field. The first reads just a single metadata from the media file and the second reads all the metadata.

#### Reading stream information
Metadata is good for reading some optional info about the media, but where do we find the important ones? You know, information like the duration of the media, how many streams the media contains, the duration of each stream in the media and a ton of other information you might want to know about the media.

I could start listing some of these things here, but it'll make this article longer than it should, and it'll not really help you with figuring things out for yourselves. If you're using an IDE (and I really hope you are), copy the first example and check the properties of the format context variable. You know, that arrow notation thing (where you'll do `ctx->...`). You'll find lots of info about the media, notably:
+ The duration of the media `ctx->duration`
+ Number of streams inside the media `ctx->nb_streams`
+ Stream-specific information (stored in an `AVStream`) `ctx->streams[index]`

One thing that is worth noting is that when it comes to things like timing and duration, things tend to be a bit confusing, so I'll try to help here.

Before you can get the duration from the media in any sane unit, you'll need to convert that value you got (from either the format context or the streams, cause streams too have their own specific duration). If you're converting the format context's duration, you'll need to divide it by `AV_TIME_BASE` to get the result in seconds. Make sure you cast one of the operands to double to get a more accurate value.
```c++
double duration = ctx->duration / (double) AV_TIME_BASE;
```
For the stream duration, you have to do a lot more. There's something called time_base stored in each of the streams contained in the format context. Let's take a look at how you would convert each of the streams' duration to seconds:

```c++
int stream_index = ...
AVStream* stream = ctx->streams[stream_index];
double duration = stream->duration * av_q2d(stream->time_base);
```
The `av_q2d` method converts the AVRational value to a double you can multiply the duration by to get your result in seconds.

**NOTE:** The time base is also useful in other situations when you need to convert pts values. We'll look at those when we start muxing and decoding stuff.

#### Reading packets
Let's learn about how to read packets from containers. First, open the media and read the stream info as in the example above. Next, we're going to sit in a loop reading (demuxing) the packets for each stream contained in our media file:
```c++
AVPacket* packet = av_packet_alloc();
while (av_read_frame(ctx, packet) == 0) {
	// Send the packet to the decoder, or do something else with it.
	// ...
	
	// After working with the packet
	av_packet_unref(packet);
}

/// Free the packet, we're done with it
av_packet_free(&packet);
```

This is essentially how we read packets from the demuxer. The `AVPacket` structure contains a field `stream_index` that specifies what stream the packet is meant for. When we talk about the `libavcodec` library, you'll see how everything comes together.
