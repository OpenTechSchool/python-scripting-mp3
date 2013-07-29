---

layout: ots
title: Scripting the MP3 Library with Python

---

# Scripting the MP3 Library with Python

This Workshop will introduce python as a scripting language for you. Scripting in here is referred to as small tasks, often used for system administration and filesystem organisation, which do not really require a full blown program but just some little script to automate tasks. In this particular case we'll be working on organsing MP3-Files depending on their embedded tags. As such it is assumed you are having a small batch of mp3 files you'd like to work with. If you got not, ask a person around you to help you out with that.

## Install and Setup

### Python
It is assumed, that you have an installation of Python 2.7 on your system. If you have a recent Mac or Linux, you are covered. In case you are running a Winows Operation System, please refer to the Setup page of the [Python on Windows](http://docs.python.org/2/using/windows.html#excursus-setting-environment-variables).

### eyeD3
We'll be using an eyeD3 Helper library to read and tamper with the tags. A library is just a bunch of code files, which encapsulate a group of tasks and are packaged togehter to be distributed. Python has plenty of libraries all kinds of tasks, almost all of them are listed on [pypi](http://pypi.python.org/pypi). The Library we'll be using is called "eyeD3".

	pip install eyed3

Test that the library is installed correctly

	$ python
	[GCC 4.2.1 (Based on Apple Inc. build 5658) (LLVM build 2335.15.00)] on darwin
	Type "help", "copyright", "credits" or "license" for more information.
	>>> import eyed3
	>>>

If you don't get an error the library is installed

## First Steps 

### Accessing the tags

Alright, let's get started then. Use the Command Line from before, staying in the same directory and start a python shell by typing `python` and hit enter. Now import the library with the following command:

	>>> import eyed3

And parse a sample file found in "samples/tagged/song1.mp3"

	>>> audiofile = eyed3.load("samples/tagged/song1.mp3")

	>>> print audiofile
	<eyed3.mp3.Mp3AudioFile object at 0x10c783090>

Now you have an eyed3 object. If you want an indepth view of the library you can read the full api documentation here [eyed3 api documentation](http://eyed3.nicfit.net/api/modules.html). To access the tag information you have to used the tag object. View the contents of the tag usign the following.

	>>> print audiofile.tag.artist
	python_teachers
	>>> print audiofile.tag.album
	open_tech_school_mp3_lesson
	>>> print audiofile.tag.title
	python_rocks_song_number_one
	>>> print audiofile.tag.track_num
	(1, None)
	>>>

Not the last field "track_num" is not returning a string. That is because according to the documentation the value returned is a "2-tuple of (track-number, total-number-of-tracks)"

You now have an object, which encapsulates the tags of that particular file for you. As such it has a bunch of properties representing the different tags in the file, as 'artist', 'album', 'title' and others.

*Excercise*: Print artist, album, title and genre of another file. If they are empty, try a different file until you have at least one file of which you know the tags now. Remember this one.

### Changing Tags
You can't only read but also write files via this library. Just assign a different value to the attribute and call the save method of the object. Lets clean up the values to nice proper cases and spaces instead of the underscore and lowercase:

	>>> audiofile.tag.artist = u"Python Teachers"
	>>> audiofile.tag.album = u"Open Tech School MP3 Lesson"
	>>> audiofile.tag.title = u"python_rocks_song_number_one"
	>>> audiofile.tag.save()

Notice the u before the strings. This is because the eyed3 library expects a unicode string.

You can re-open the file and print the value to verify it worked:

	>>> audiofile = eyed3.load("samples/tagged/song1.mp3")
	>>> print audiofile.tag.artist
	Python Teachers


Congratulations, you just changed the ID3-Tags of that file. And it works the same way on all other files as well.

## File Renamer
Now, let's make ourself a little script that renames the file depending on its tags. For that we also need the build-in "os" library, which exposes a function called "rename", with the two parameters of the current filename and the new filename. So after you `import os`, you need to compile the filename from the tags like this:

	new_filename = "samples/tagged/{0}-{1}.mp3".format(audiofile.tag.artist, audiofile.tag.title)

Once we have done that, we can use "os.rename" to rename the file:

	os.rename('samples/tagged/song1.mp3', new_filename)

You can now see that the file has been renamed. Of course to access its tags again, you now need to use the `eyed3.load` command with that new file first.

*Excercise*: rename another file to the schema "Artist-Album-Track-Title".


## Make it into a Script

In order to have a real script, let's put this into a file now. Open your favourite editor and paste the following in there:

	import os
	import eyed3

	audiofile = eyed3.load("samples/tagged/song1.mp3")
	new_filename = "samples/tagged/{0}-{1}.mp3".format(audiofile.tag.artist, audiofile.tag.title)
	os.rename('samples/tagged/song1.mp3', new_filename)

If you now save it in the same directory under the name "renamer.py" and call it via `python renamer.py`, it will rename the file. But it will only do it once because then the file has already been renamed. So let's make it a bit more flexible by giving it a filename via the commandline. In order to do that, we'll be using the great [argparse module](http://docs.python.org/2/library/argparse.html#module-argparse) that gets shipped with python. Just put this at the top of your file (after the other imports):

	import argparse

	parser = argparse.ArgumentParser()
	parser.add_argument("filename")

	args = parser.parse_args()

If you now execute the file again, you'll notice that it quits and tells you:

	usage: [-h] filename
	: error: too few arguments

That is the argument parser we've just set up, stopping execution at parse_args because the argument "filename" is missing on the commandline. Great, exactly what we want. Now we also need replace every place where we have the static filename right now with the new filename. We can access it as "filename" on the args that got passed. After that your file should look something like this:

	import os
	import eyed3
	import argparse

	parser = argparse.ArgumentParser()
	parser.add_argument("filename")

	args = parser.parse_args()

	audiofile = eyed3.load(args.filename)
	new_filename = "samples/tagged/{0}-{1}-{2}.mp3".format(audiofile.tag.artist, audiofile.tag.album, audiofile.tag.title)
	os.rename(args.filename, new_filename)

And if you execute it now, it should rename any given file for you depending on its tags. Try it :) .


*Excercise*: Using the files in samples/untagged do the reverse and take a file name and add tags to an mp3 with no tag information.
