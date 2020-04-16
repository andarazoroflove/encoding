Changes in alpha 0.31 (16th october, 2002)

 - Added bicubic vertical (only) resize for YV12. Works only for "Precise" and "Neutral". It runs 15+% faster than YUY2 on a P3-733 with my Excalibur test vob (720*576 to 720*432). It was not very seriously tested; let me know if you find bugs ...
 

Changes in alpha 0.3 (14th october, 2002)

 - Bug fix : virtualdubmpg2 crashed sometimes when seeking to the last frame. This should be fixed.
 
 - Bug fix : .d2v was generated twice, taking twice the time. That's why you had to wait the same amount of time again after the counter had hit 100%. As far as I can remember, the very first releases didn't have this bug.
 
 - Bug fix : audio files are no longer generated in the exe's directory but in the same directory as the input files.
 
 - Bug fix : B and P frames were reported as P and B frames in the status bar.
 
 - Added .vob support. This allows to open MPEG2 files without SmartRipper info files. The file extension MUST be ".vob"; ".mpg" will NOT work with MPEG-2 and report a "pack syncronization error" (it's far easier for me to code a strcmp(.,".vob") than trying to figure out if a file is MPEG-1 or MPEG-2 ...). This support is limited to one file. If you want to concatenate vob files, read the FAQ about smartripper .txt files and so on... Vob support includes drag and drop into virtualdubmpg2.


Changes in alpha 0.23 (10th october, 2002)

 - Bug fix : virtualdubmpg2 used to crash when selecting audio streams in a particular order. This should be fixed.


Changes in alpha 0.22 (7th october, 2002)

 - Bug fix : when seeking in virtualdub after the end of the first .vob file, frames appeared to come again from the first file. This problem did not appear when streaming from the beginning.

 - Vertical clip implemented for YV12. Not fully tested.

 - I'm implementing a custom bicubic filter. It is now only available with fixed values (neutral bicubic).
 
 
Changes in alpha 0.21 (5th october, 2002)

 - Bug fix (regression introduced in alpha 0.2) : Frame count was doubled. A movie appeared to be duplicated after its end. Should work as before now.
 
 - Added possibility to abort .d2v generation. This is not recommended.


Changes in alpha 0.2 (4th october, 2002)

 - Minor tweaks in preview trackbars: width is now a multiple of 8, height position is now correct at startup

 - Added bilinear vertical resize for YV12. On my test vob, it is 21 % faster than YUY2 (yes 21% !!! but my test vob is rather short, I'll try to do more realistic tests on longer vobs when I have time). This mode is automatically activated when 1) width is a multiple of 16, 2) there's no horizontal resize and 3) bilinear resize is chosen. Horizontal clip *works* (does anyone use it, anyway ?): when you put too high values for clip left and/or right, top and bottom begin to increase. DVD2avi's clip&resize already had this "feature"...

 - The .d2v file now remembers the filter type (it used to default to bilinear when you reloaded a file)
 
 - Job support now works even if the .d2v file was not previously generated. This is of no use for a standard usage of the program but can ease the use of VirtualDubMpg2 by another program. Note that there's no progress bar during .d2v generation, and that VirtualDubMpg2 appears to be not responding. This is normal. VirtualDub's status window will popup when .d2v creation is complete.

 - Bug fix (regression introduced in 0.11) : files could not be opened when .d2v did not already exist. Strange that only one person saw that. I was not aware of that problem myself since I always work on the same test .vob file...
 

Changes in alpha 0.11 (20th september, 2002)

 - Job support should be working now.

 - Fixed a bug where the avi output was unreadable when the user had not done any seeking at all with trackbars before saving an .avi



Changes in alpha 0.1 (18th september, 2002)

 - Added partial audio support: external files are generated with the .d2v. Works for AC3 for demuxing and decoding. LPCM untested. MPA may work if a single MPA stream is selected.
 
 - Added support for YV12 color mode (activated only when there's no clip and no resize). Performance is 7% better than YUY2 on my Excalibur test vob (720*576 PAL, 4718 frames) with DivX 5.0.2 Pro, B-Frames and GMC activated.








Old readme.txt :



VirtualDubMpg2 basically adds AC3,MP3 and MPEG2 support to VirtualDub 1.4.10.

See http://www.virtualdubmpg2.fr.st for new releases.


VirtualDubMpg2 is based upon the work of the following people:

Avery Lee (http://www.virtualdub.org) for VirtualDub 1.4.10
Nando (Has he a website somewhere ?) for Nandub 1.0RC2
Jackei (http://arbor.ee.ntu.edu.tw/~jackei/dvd2avi/) for DVD2avi
	Actually, i've taken the DVD2AVIT3 version from trbarry that includes the OGO modifications.
	
	


Known bugs:

- One (just one) frame is often "forgotten" during a conversion.
- the very last frames of the stream may get corrupted
- do not use "display input video" when dubbing with normal recompression mode. You may get a distorted picture (you'll see what happens when one tries to display as RGB a buffer containing YUV2 data...)
- If you open several smartripper info files in a single virtualdub session, you'll only obtain images from the first one (due to dirty programming with global variables that do not get reset...)
- and some more ...



Known limitations:

- VirtualDubMpg2 only opens smartripper files (version 2.34, I did not test above), not single vob files. Just write a txt file by hand from an existing one by changing the interesting data (name, streams,vob paths)if you don't have one.
- No sound for the moment (). Well, in fact you may (it depends on the stream number) obtain a wav file during the d2v file generation, but that's not done on purpose for the moment :).
- Job support still not totally functional (okay okay I must confess it does not work at all..)



Installation

to limit the distribution size, required dlls are not given with the exe. You'll need the ones from VirtualDub 1.4.10 and Nandub 1.0RC2 in the exe's folder.
