# VNC Aligner for macOS

Align confocal images of the fly ventral nerve cord (VNC) to the JRC2018 unisex VNC template. The program produces aligned 3D images and color-depth projections for viewing your labeled neurons.

By Hideo Otsuna. Fiji, the required plugins, alignment tools, and templates are included.

## Before you start

You need a Mac and an `.lsm` image stack with **2–4 channels**: one anatomical reference and up to three signal channels, such as labeled neurons.

**The program automatically selects the reference channel based on signal amount.** It compares all input channels across the image stack and uses the channel with the most signal as the reference. The reference can be in any channel position. Check that the selected channel is your anatomical reference when inspecting the result.

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

The `.sh` script sets the Fiji location and selects its bundled Java automatically.

Run this command, replacing both example paths with your own locations:

```bash
bash 20x40xVNC_Align_CMTK_localMac_LSF.sh \
    "/full/path/to/confocal_stack.lsm" \
    8 \
    "/full/path/to/Results_run1"
```

The first path must be the **full file path to your confocal stack**, starting with `/`. Dragging the stack from Finder into Terminal inserts its full path. The last path is where you want the results saved; the program creates that folder.

Leave `8` as the example number of processing threads for your first run. Press **Return** to start.

Messages will appear in Terminal while the image is being processed. Keep Terminal open until the command finishes.

**For another run, change `Results_run1` to a new folder name**, such as `Results_run2`. This prevents the program from reusing an earlier or incomplete result.

## 4. Find and check your results

For an input called `your_image.lsm`, look inside `Results_run1`:

| File | What it contains |
| --- | --- |
| `your_image/your_image_01.nrrd` | Aligned reference selected by the program |
| `your_image/your_image_02.nrrd` | First aligned signal channel |
| `your_image/your_image_03.nrrd` and `your_image/your_image_04.nrrd` | Additional aligned signal channels, when present in the input |
| `MIP/` | Color-depth projections for each output channel |

Output `_01` is always the selected reference. The remaining channels are saved as `_02`, `_03`, and `_04` in their original input order, skipping the reference. A two-channel input produces two aligned volumes; a four-channel input produces four.

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
