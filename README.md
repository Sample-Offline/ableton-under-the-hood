# Ableton "Under-the-hood"

Some commands, tips, tools, and other magics for working with Ableton... everywhere except inside the DAW.

## Overview

I've had my fair share of problems using Ableton over the years, and this is an attempt to quell some of those frustrations and the ever-present anxiety that comes from moving projects around. Do I [Ctrl] + [Shift] + S to save?, or do I "Save as a Copy...?" _or_, do I create a pack? Do I **Collect All and Save**? Even the Factory Packs? Why does the File Manager still show uncollected files even if I have the "Collect All and Save" option checked in the Preferences?

If not outright solutions, hopefully these will help you avoid Sample Offline errors, missing files, and other errata. Maybe there's even a tip or two for all the modding folks out there.

Enjoy!
... or don't. Who am I, the enjoyment police?

## 1. Working with .als files

You can convert an `.als` file to `.xml` using **gzip**, then convert it back to `.als` when you need to open it in Ableton. This can be useful for a number of reasons:

1. You can version control the xml without using git LFS
2. You can manually `sed` or find/replace file paths (more on this below)
3. You can batch process the xml files using a custom script or whatever. which might be faster than opening the File Manager and doing it per-project.
   1. Have it crawl through the projects folder, converting `.als` to `.xml` whenever it encounters one, then add 'em all to **git** and then have the script convert them back.

```bash
# Convert
gzip -cd MyProject.als > MyProject.xml

# After doing whatever with the xml, you can convert it back.
# Note that in my experience, you need to use this method, otherwise Ableton may bark about a corrupted project file.
gzip -S .als MyProject.xml > MyProject.als

```

Though, you may want to do some file massaging beforehand. There might be a better way to do this, but...

```bash
```bash
# Convert
gzip -cd MyProject.als > MyProject.xml

# Note this preserves the `.als` file in the dir. Maybe make a backup if you're paranoid, or just yolo it.
# This simply moves it to the orig/ dir where you can then keep/delete it after you're done.
mkdir -p orig && mv MyProject.als orig/

# After doing whatever with the xml, you can convert it back.
# Note that in my experience, you need to use this method, otherwise Ableton may bark about a corrupted project file.
gzip -S .als MyProject.xml > MyProject.als

# Im too lazy to google how to get rid of the existing ext when using gzip, so I just move the file. Voilá.
mv MyProject.xml.als MyProject.als

# Check the file (This is the file that was converted to .xml then back to .als)
gzip -lv MyProject.als

# This should output something similar to:
#    method  crc     date  time    compressed uncompressed  ratio uncompressed_name
#    defla 13a905ca Sep 16 01:48         5964        52805  88.7% MyProject.als

# If it opens in Ableton, and everything looks good, you can remove the original copy
rm -rf orig/
```

_example of `pop_vox.als` xml file's sample reference:_

```xml
<SampleRef>
        <FileRef>
                <RelativePathType Value="3" />
                <RelativePath Value="Samples/Processed/Consolidate/#1 POP STAR, only session [2023-09-16 011042].wav" />
                <Path Value="/Users/pseudo/Desktop/pop_vox Project/Samples/Processed/Consolidate/#1 POP STAR, only session [2023-09-16 011042].wav" />
                <Type Value="2" />
                <LivePackName Value="" />
                <LivePackId Value="" />
                <OriginalFileSize Value="4992044" />
                <OriginalCrc Value="35868" />
        </FileRef>
        <LastModDate Value="1694844696" />
        <SourceContext />
        <SampleUsageHint Value="0" />
        <DefaultDuration Value="1248000" />
        <DefaultSampleRate Value="48000" />
</SampleRef>
```

If you ever needed to find and replace the path to a sample manually (for whatever reason), you can search for the filename, then change the line with the `<Path Value>` attribute to the new file path.

## 2. Fonts

If you want the fonts used by the app, for whatever reason, they're at:
`/Applications/Ableton Live 11 Suite.app/Contents/App-Resources/Fonts`. You can then install them to the OS using Font Book or whatever the Windows one is. (I know, the TODO is to make OS-agnostic paths for all this).

As of Ableton 11, the list should look similar to:

```plaintext
AbletonSans-Light.ttf
AbletonSansSmall-Bold.ttf
AbletonSansSmall-Regular.ttf
AbletonSansSmall-RegularItalic.ttf
NotoSansCJKjp-Bold.otf
NotoSansCJKjp-Regular.otf
NotoSansCJKkr-Bold.otf
NotoSansCJKkr-Regular.otf
NotoSansCJKsc-Bold.otf
NotoSansCJKsc-Regular.otf
unifont_jp-14.0.01.ttf
unifont_upper-14.0.01.ttf
```

> Note: Using these for anything other than personal voyeurism is most likely against the TOS, so don't get yerself sued.

## 3. Internal Ableton reference Databases

The file reference map db, conveniently called `filerefmap.db`
is at `/Applications/Ableton\ Live\ 11\ Suite.app/Contents/App-Resources/Database/filerefmap.db`. You can query it using sqlite:

```bash
cp /Applications/Ableton\ Live\ 11\ Suite.app/Contents/App-Resources/Database/filerefmap.db ~/tmp/

sqlite3 ~/tmp/filerefmap.db

sqlite> select * from pack_names;
# www.ableton.com/95|Session Drums Club
# www.ableton.com/94|Session Drums Studio
# www.ableton.com/78|Soniccouture Pan Drum
# www.ableton.com/91|Grand Piano
# www.ableton.com/5|Orchestral Strings
# www.ableton.com/93|EIC2 Legacy
# www.ableton.com/65|SonArte Tubular Bells
# www.ableton.com/ORCHESTRAL_BRASS_LEGACY|Orchestral Brass Legacy
# www.ableton.com/119|EIC2 Legacy Orchestral
# www.ableton.com/118|Session Drums LE Legacy
# www.ableton.com/96|Soniccouture Electric Pianos
# www.ableton.com/ORCHESTRAL_MALLETS_LEGACY|Orchestral Mallets Legacy
# www.ableton.com/Live8LegacyNEW|Live 8 Legacy Library NEW
# www.ableton.com/113|DM ARP 2600 Drums by Flatpack
# www.ableton.com/112|Breakbeats by KutMasta Kurt
# www.ableton.com/115|Flatpack Texture Beats
# www.ableton.com/114|Sample Logic Tronix
# www.ableton.com/117|EIC2 LE Legacy
# www.ableton.com/116|Sample Magic Radio Slave for Live
# www.ableton.com/51|e-instruments Studio Grand Downtown
# www.ableton.com/50|AAS Journeys
# etc...
# etc...
```

```bash
# The other table of note is `sample_mappings`
sqlite> select * from sample_mappings;

# WHich gives you a bunch of internal paths like:
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - C4.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - C4.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - C5.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - C5.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - C6.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - C6.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - F#0.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - F#0.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - F#1.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - F#1.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - F#2.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - F#2.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - F#3.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - F#3.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - F#4.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - F#4.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Phantasm/Phantasm - F#5.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Phantasm/Phantasm - F#5.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Limette/Limette f1.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Limette/Limette f1.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Limette/Limette f2.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Limette/Limette f2.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Limette/Limette f3.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Limette/Limette f3.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Limette/Limette f4.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Limette/Limette f4.aif
# 5|www.ableton.com/0|Samples/Ambient & Evolving/Limette/Limette f5.aif|5|www.ableton.com/0|Samples/Multisamples/Synths/Limette/Limette f5.aif
# 5|www.ableton.com/0|Samples/Drums/Electronic Percussion/Perc Turn Off The TV.wav|5|www.ableton.com/0|Samples/One Shots/Drums/FX Hit/FX Turn Off TV.wav
# 5|www.ableton.com/0|Samples/Drums/Electronic Percussion/Perc Turn On The TV.wav|5|www.ableton.com/0|Samples/One Shots/Drums/FX Hit/FX Turn On TV.wav
# 5|www.ableton.com/0|Samples/Drums/Wood/Perc Wood Feel.wav|5|www.ableton.com/0|Samples/One Shots/Drums/Wood/Wood Feel.wav
# 5|www.ableton.com/0|Samples/Drums/Wood/Resonant Knock.wav|5|www.ableton.com/0|Samples/One Shots/Drums/Wood/Wood Resonant Knock.wav
# 5|www.ableton.com/0|Samples/Drums/Rim/Strike Kindified.wav|5|www.ableton.com/0|Samples/One Shots/Drums/Snare/Snare Strike Kindified.wav
# 5|www.ableton.com/270|CV Instruments/CV Instrument.amxd|5|www.ableton.com/270|CV Instruments/CV Instrument/Ableton Folder Info/CV Instrument.amxd
# etc...
# etc...
```

## 3. Custom Metronomes

TODO

## 4. Theming

TODO
