# VNC aligner files

Collected on October 19, 2026 from the installation used successfully for sample, Goro, I3, W4, and W5. Copied dependencies were verified against their sources using SHA-256. The main shell script was subsequently updated to select the 2026 macro and use this package's paths. The 2026 macro was subsequently corrected to restore the height-normalized depth search. Previous versions are saved as timestamped .bak files. COPY_MANIFEST.json and SHA256SUMS.txt record the updated script and macro checksums.

As of October 27, 2026, the packaged main script hard-codes `VNC_DV_FLIP=off` for every sample. Environment values cannot override it; use plain `bash SCRIPT FILE_PATH CPU_COUNT OUTPUT_FOLDER` commands. It no longer creates `FinalOutputs`; each sample's output folder contains its debug logs and any error properties. The documented sample no-flip run scored 0.8818 (2022: 0.8578); the earlier 0.8889 result had reversed DV orientation and is unsuitable for that sample. 

## Contents

- `20x40xVNC_Align_CMTK_localMac_LSF.sh`: current main script, configured for this Mac with the 2026 preprocessing macro, hard-coded DV flip off, and per-sample debug logs.
- `align_run_tested.sh`: exact script used for the Diego batch, with the preprocessing macro path corrected to `OLD/` and separate per-sample debug logs.
- `OLD/VNC_preImageProcessing_Pipeline_07_29_2022.ijm`: preprocessing macro actually used in all five successful runs.
- `VNC_preImageProcessing_Pipeline_05_22_2026.ijm`: active macro in both copies of the main script. Includes the depth-search correction and optional DV flip control; .
- `nrrd_compression.ijm`, `Score_Generator_Cluster.ijm`, and `Color_Depth_MIP_batch_0404_2019_For_Pipeline_packbits.ijm`: output compression, scoring, and color-depth projections.
- `Template_MIP/`: VNC projection masks, including unisex, female, male, and legacy VNC variants. These are projection masks, not the excluded registration template folder.
- `For_Score/`: masks referenced by the shell script; the tested scoring macro currently uses `UseMask=false`.
- `Fiji_plugins/`: installed custom plugin files from `Plugins_Color_MIP`, `63x_fiji_plugins`, the root plugins directory, and `EM_based_seg_CDM_plugins`. This includes NRRD writing, alignment scoring, image correlation, gamma, segmentation, rotation, color-depth projection, PackBits saving, and the quit plugin.
- `Fiji_luts/`: PsychedelicRainBow2 LUT from the working Fiji installation.
- `cmtk/`: the four macOS CMTK 3.3.1 executables invoked by this pipeline.

## External files and software

`VNC_Template/` is deliberately NOT copied or linked here. The existing folder is:

    /Users/otsunah/test/VNC_aligner_local_ver/VNC_Template

The tested settings require `JRC2018_VNC_UNISEX_1micron.nrrd`, `JRC2018_VNC_UNISEX_447_G15.nrrd`, and the lateral-reference images used by preprocessing (`VNC_Lateral_M.tif` / `VNC_Lateral_F.tif`) from that external folder.

A complete Fiji installation with Java is still required; the Fiji application and Java runtime are not included. The working installation is:

    /Users/otsunah/Downloads/Fiji_20221005-1117..app

The bundle contains the custom add-on plugins. Standard Fiji features (Bio-Formats, NRRD reading through Fiji IO, CLAHE/mpicbg, Anisotropic Diffusion 2D, and core ImageJ operations) are supplied by the Fiji installation and its libraries. Optional V3Draw paths additionally require Fiji's Vaa3D plugins. The installed Java used in the successful runs was Zulu Java 8, version 1.8.0_322. The included CMTK executables are macOS x86_64 binaries, not Linux or Windows builds.

## Installation and paths

For another Fiji installation, merge the contents of `Fiji_plugins/` into its `plugins/` directory, preserving the subdirectories, and copy `Fiji_luts/PsychedelicRainBow2.lut` into its `luts/` directory. Restart Fiji. Use one installed copy of each custom plugin class to avoid ambiguous duplicate versions.

The current main script is configured for this Mac. `align_run_tested.sh` is a historical runner from the older Diego batch and still selects the 2022 macro. To relocate the package, edit the local/test-mode configuration in `20x40xVNC_Align_CMTK_localMac_LSF.sh`:

1. Set `MACRO_DIR` to this bundle's absolute path, with a trailing `/`.
2. Set `FIJI` to the chosen Fiji executable.
3. Set `CMTK` to this bundle's `cmtk/` directory.
4. Set `TempDir` to the external `VNC_Template/` directory; the packaged script currently uses its absolute path on this Mac.
5. Leave `MIPTempDir` pointing to this bundle's `Template_MIP/` and `PREPROCIMG` pointing to its `VNC_preImageProcessing_Pipeline_05_22_2026.ijm` at the package root.

Invocation takes three arguments:

    bash 20x40xVNC_Align_CMTK_localMac_LSF.sh /absolute/path/sample.lsm 8 /absolute/path/output

Use absolute paths without spaces or commas; these macros and the original shell script do not consistently support such paths. The current test uses 8 threads, template number 1, gender `u`, and objective `40x`. With the local `sr` channel specification, the 2026 macro forces channel 2 as reference and forces shape analysis off internally. Changing the shell nc82decision setting alone does not override this. The five earlier runs used the 2022 macro and automatic reference selection. No sample images or alignment output volumes were copied into this bundle. pdf for setup and usage.

## DV orientation and simple commands

The packaged main script contains `VNC_DV_FLIP=off`. It always passes `off` as argument 11 to the macro, which reads it from `args[10]`. CPU count is `args[7]` in this 11-argument layout. The separate 12-argument voxel-override layout uses `args[10]` for CPU count; the two layouts do not conflict.

`off` excludes flipped lateral fitting candidates and prevents the corresponding DV flip in reference and signal. Other orientation processing is unchanged. To change this fixed setting, edit the packaged script itself; an environment prefix cannot override it. Other copies of the script may have different settings.

After the Java setup in the manual, use the following from Terminal:

```bash
cd /Users/otsunah/test/VNC_aligner_local_ver/Necessary_files_VNC_Aligner
out="/Users/otsunah/test/VNC_aligner_local_ver/output_I3_$(date +%Y%m%d_%H%M%S)"
for file in /Users/otsunah/test/VNC_aligner_local_ver/sample/Diego/I3/*.lsm; do
    [ -f "$file" ] || continue
    bash 20x40xVNC_Align_CMTK_localMac_LSF.sh "$file" 8 "$out"
done
```

The loop is sequential and does not recurse into subfolders. No environment prefix or filename-specific flip logic is required. `run_sample_no_flip.sh` remains an optional convenience launcher for its fixed sample input and Java setup; it is not required to disable flipping. Use fresh output folders to avoid cached results. Confirm `DV flip mode: off` in each `debug/preproc.log` and inspect the final images.
