# Ableton "Under-the-hood"

Some commands, tips, tools, and other magics for working with Ableton... everywhere except inside the DAW.

## Overview

I've had my fair share of problems using Ableton over the years, and this is an attempt to quell some of those frustrations and the ever-present anxiety that comes from moving projects around. Do I [Ctrl] + [Shift] + S to save?, or do I "Save as a Copy...?" _or_, do I create a pack? Do I **Collect All and Save**? Even the Factory Packs? Why does the File Manager still show uncollected files even if I have the "Collect All and Save" option checked in the Preferences?

If not outright solutions, hopefully these will help you avoid Sample Offline errors, missing files, and other errata. Maybe there's even a tip or two for all the modding folks out there.

Enjoy!
... or don't. Who am I, the enjoyment police?

## 1. Working with .als files

### 1.1 Manually fixing missing samples

**Scenario:** So you have a project, but your lazy-ass saved it on the Desktop.  You **Collect All And Save**, and move on to the next one. You do this for years. And years. Until you realize the Desktop is probably a bad place to store the files. How do you move them? Will all your projects be FUBAR'd?

### 1.2 Version Control

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
