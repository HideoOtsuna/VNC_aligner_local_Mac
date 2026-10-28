# VNC Aligner for macOS

Align confocal images of the fly ventral nerve cord (VNC) to the JRC2018 unisex VNC template. The program produces aligned 3D images and color-depth projections for viewing your labeled neurons.

By Hideo Otsuna. Fiji, the required plugins, alignment tools, and templates are included.

## Before you start

You need a Mac and a two-channel `.lsm` image stack containing an anatomical reference and the signal you want to align, such as labeled neurons.

**The program automatically selects the reference channel based on signal amount.** It compares the signal across the image stack and uses the channel with more signal as the reference. Either channel order can be used. Check that the selected channel is your anatomical reference when inspecting the result.

Use file and folder names without spaces or commas. Start with one image and inspect the result before processing a full experiment.

## 1. Download the package

1. Install [GitHub Desktop](https://desktop.github.com/download/).
2. Open it and choose **File > Clone Repository > URL**. “Clone” means download a copy to your computer.
3. Paste this address:

   ```text
   https://github.com/HideoOtsuna/VNC_aligner_local_Mac
   ```

4. Choose where to save it, keep the folder name `VNC_aligner_local_Mac`, and click **Clone**.
5. Wait for the download to finish. The package includes several large files.

Use GitHub Desktop for this download. The website's **Download ZIP** button may leave out the large Fiji and template files.

## 2. Unpack Fiji

In Finder, open the downloaded `VNC_aligner_local_Mac` folder and double-click **`Fiji_VNC_aligner.zip`**.

After it expands, you should see **`Fiji_VNC_aligner.app`** in the same folder. Keep it there, together with the `Template` and `Copy_to_other_Fiji` folders. The program uses these locations automatically.

## 3. Run your first image

Open **Terminal** from **Applications > Utilities**.

Type `cd` followed by a space, drag the downloaded `VNC_aligner_local_Mac` folder from Finder into Terminal, and press **Return**. This tells Terminal which folder to work in.

Copy and paste these two lines into Terminal, then press **Return**. They select the Java included with Fiji. Repeat this step whenever you open a new Terminal session for alignment.

```bash
export JAVA_HOME="$PWD/Fiji_VNC_aligner.app/java/macosx/adoptopenjdk-8.jdk/jre/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
```

Next, choose the image to align:

1. Type `image=` in Terminal, with no spaces.
2. Drag your `.lsm` file from Finder into Terminal, then press **Return**.

Now copy and paste this command and press **Return**:

```bash
bash 20x40xVNC_Align_CMTK_localMac_LSF.sh "$image" 8 "$PWD/Results_run1"
```

Leave `8` as the example number of processing threads for your first run. The program creates a results folder called **`Results_run1`** inside the downloaded project folder.

Messages will appear in Terminal while the image is being processed. Keep Terminal open until the command finishes.

**For another run, change `Results_run1` to a new folder name**, such as `Results_run2`. This prevents the program from reusing an earlier or incomplete result.

## 4. Find and check your results

For an input called `your_image.lsm`, look inside `Results_run1`:

| File | What it contains |
| --- | --- |
| `your_image/your_image_01.nrrd` | Aligned reference selected by the program |
| `your_image/your_image_02.nrrd` | Aligned image from the other input channel |
| `MIP/your_image_01.tif` and `MIP/your_image_02.tif` | Color-depth projections |

Open the `.nrrd` images in Fiji. Compare the aligned reference with `Template/JRC2018_VNC_UNISEX_447_G15.nrrd`, and check the shape and orientation of the VNC before using the aligned signal for analysis.

The selected input channel is recorded in `your_image/debug/preproc.log`, together with the signal amount measured in each channel.

An alignment score is also saved in the image's results folder. **A high score alone does not guarantee correct anatomical alignment or orientation.**

## If something goes wrong

| Problem | What to check |
| --- | --- |
| Fiji ZIP will not expand, or a template opens as a few lines of text | Download the package with GitHub Desktop and let the large files finish downloading. |
| The program cannot find Fiji | Confirm that `Fiji_VNC_aligner.app` is inside the downloaded project folder. |
| The run stops or the images look wrong | Check which channel was selected as the reference and read the messages in Terminal. Use a new results folder when trying again. |

For batch processing, computer compatibility, settings, and detailed troubleshooting, see the [technical guide](docs/Technical_guide.md).

The bundled application has had its dependencies checked; a new full alignment has not yet been run with this packaged version. Validate a representative image from your experiment before running a batch.

See [LICENSE](LICENSE) for licensing information.
