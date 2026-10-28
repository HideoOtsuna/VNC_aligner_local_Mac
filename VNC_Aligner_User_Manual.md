# VNC Aligner: setup and usage manual

For `20x40xVNC_Align_CMTK_localMac_LSF.sh` on macOS

Updated October 27, 2026 for the packaged script: hard-coded DV flip off, simple Bash commands, and the current output layout.

## 1. Start here

This script preprocesses a VNC image in Fiji, registers it with CMTK, resamples its channels into a reference template, calculates an alignment score, and creates color-depth projections.

The current configuration is **local mode, 8 threads, unisex template number 1, and channel 2 as reference**. The active preprocessing macro is the 2026 version; earlier validation used the 2022 version. Other input formats, 20x acquisition, and other template modes require separate validation.

**DV orientation:** the packaged script hard-codes `VNC_DV_FLIP=off` for every sample. No environment prefix or filename exception is needed. Inspect reference anatomy and slice direction after alignment; a high score alone does not establish correct orientation.

**Use the script inside `Necessary_files_VNC_Aligner`.** This manual describes that copy and its 2026 macro, configured for this Mac. Other copies may have different settings. `align_run_tested.sh` is a historical runner that still uses the 2022 macro.

### Files and folders

The examples use these locations:

```bash
BASE=/Users/otsunah/test/VNC_aligner_local_ver
PKG="$BASE/Necessary_files_VNC_Aligner"
```

| Location | Purpose |
| --- | --- |
| `$PKG/20x40xVNC_Align_CMTK_localMac_LSF.sh` | Script to configure and run |
| `$PKG/VNC_preImageProcessing_Pipeline_05_22_2026.ijm` | Active preprocessing macro |
| `$PKG/Template_MIP/` | VNC masks for projection generation |
| `$PKG/Fiji_plugins/` and `Fiji_luts/` | Custom Fiji add-ons and color LUT |
| `$PKG/cmtk/` | Four CMTK executables used by the pipeline |
| `$BASE/VNC_Template/` | External registration templates and lateral images |

**VNC_Template was intentionally excluded from the package.** Keep it outside the package and set `TempDir` to its actual location. Do not confuse `VNC_Template` with the included `Template_MIP` masks.

Use a full Fiji installation with working Java, Bio-Formats, Fiji IO/NRRD reading, CLAHE, and the custom plugins. The copied CMTK executables are macOS x86_64 builds, version 3.3.1. Their operation was verified on this Mac; another computer needs compatible executables and Java.

Use absolute paths without spaces or commas for the package, Fiji, templates, inputs, and outputs. Quoting your Terminal command does not fix all unquoted paths or comma-separated macro arguments inside this script.

---

## 2. Configure the shell script

The packaged `20x40xVNC_Align_CMTK_localMac_LSF.sh` already has the settings below. To customize it, open it in a plain-text/code editor and save a backup. Keep `testmode=1` near the top: here, “Test mode” means **run locally**, not a dry run. It performs the complete alignment.

Inside `if [[ $testmode == "1" ]]; then`, check these assignments. If relocating the package, replace the paths in place; do not leave later assignments that overwrite them.

```bash
MACRO_DIR="/Users/otsunah/test/VNC_aligner_local_ver/Necessary_files_VNC_Aligner/"
FIJI=/Users/otsunah/Downloads/Fiji_20221005-1117..app/Contents/MacOS/ImageJ-macosx
CMTK="${MACRO_DIR}cmtk"

TEMPNUMBER=1
INPUT1_GENDER="u"
objective="40x"

TempDir="/Users/otsunah/test/VNC_aligner_local_ver/VNC_Template"
MIPTempDir="${MACRO_DIR}Template_MIP"
PREPROCIMG="${MACRO_DIR%/}/VNC_preImageProcessing_Pipeline_05_22_2026.ijm"
NRRDCOMP="${MACRO_DIR}nrrd_compression.ijm"
```

The trailing `/` in `MACRO_DIR` matters because the script concatenates filenames onto it. `FIJI` must name the executable, not just the `.app` folder. `CMTK` must name the directory containing `registration`, `warp`, `reformatx`, and `make_initial_affine`.

The main script selects the 2026 macro with the tested depth-search correction. The 2022 macro remains under `OLD/` for historical reproducibility. These macros differ in preprocessing behavior; use a fresh output directory when switching versions.

### DV flip is fixed off

Near the top of the packaged script:

```bash
VNC_DV_FLIP=off
```

This assignment overrides any incoming environment value, including `VNC_DV_FLIP=auto`. Use plain `bash` commands. The script passes `off` as macro argument 11 (`args[10]`); CPU count is `args[7]` in that layout. The separate 12-argument voxel-override layout uses `args[10]` for CPU count, so those assignments do not run together.

This disables automatic lateral DV flipping in reference and signal. Other orientation processing is unchanged; inspect the result if an input is inverted in another way.

### Preprocessing logs and output

`DEBUG_DIR="${OUTPUT}/debug"` stores each sample's preprocessing log in its output folder. `FinalOutputs` is no longer created. Error properties, when generated, also go into the sample output folder.

### Select the reference channel

The local shell command passes `sr`. The 2026 macro maps `sr` to channel 2 and overrides the shell's `nc82decision="Signal_amount"`. Changing only that shell variable will not change this selection. Confirm `ch2 is reference` in `preproc.log` and use this configuration for samples whose reference is channel 2. A different channel layout requires coordinated macro/call changes and validation. The earlier 2022 macro used automatic signal-based selection.

---

## 3. Set up Fiji, Java, and templates

### Fiji plugins

The working Fiji installation already has the required plugins. No reinstallation is needed when using that same application. For another Fiji installation, merge the **contents** of `Fiji_plugins/` into its `plugins/` folder, preserving the subfolders. Do not place an extra `Fiji_plugins` wrapper folder inside `plugins`.

```text
Fiji.app/plugins/Plugins_Color_MIP/...
Fiji.app/plugins/63x_fiji_plugins/...
Fiji.app/plugins/EM_based_seg_CDM_plugins/quit_plugin.jar
Fiji.app/plugins/Image_Correlation_Atomic_EQ.jar
Fiji.app/plugins/Max_value.jar
Fiji.app/luts/PsychedelicRainBow2.lut
```

Copy the LUT from `Fiji_luts/` and restart Fiji. Check for commands including `Nrrd Writer`, `Alignment Scores`, `Image Correlation Atomic EQ`, `Gamma samewindow noswing`, `PackBits save activeIMG headless`, and `quit plugin`. Avoid installing conflicting versions of the same custom plugin. Standard Fiji and its libraries are separate from these custom add-ons.

### Java on this Mac

Run these commands in each Terminal session before launching the script. This is the bundled Java used for the successful runs; change the path on another computer.

```bash
JAVA_BUNDLE=/Users/otsunah/Downloads/Fiji_20221005-1117..app/java/macosx
export JAVA_HOME="$JAVA_BUNDLE/zulu8.60.0.21-ca-fx-jdk8.0.322-macosx_x64/jre/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
"$JAVA_HOME/bin/java" -version
```

The verified Java version was `1.8.0_322`. Some Fiji launcher messages said Java was not found even during successful runs. Confirm that the macro actually starts and that outputs are produced; a startup message alone is not a reliable diagnosis.

### Before the first run

```bash
BASE=/Users/otsunah/test/VNC_aligner_local_ver
PKG="$BASE/Necessary_files_VNC_Aligner"
SCRIPT="$PKG/20x40xVNC_Align_CMTK_localMac_LSF.sh"
bash -n "$SCRIPT"
"$PKG/cmtk/registration" --version
ls "$BASE/VNC_Template/JRC2018_VNC_UNISEX_1micron.nrrd"
ls "$BASE/VNC_Template/JRC2018_VNC_UNISEX_447_G15.nrrd"
ls "$BASE/VNC_Template/VNC_Lateral_M.tif"
ls "$BASE/VNC_Template/VNC_Lateral_F.tif"
```

`bash -n` should produce no output and exit successfully. CMTK should print `3.3.1`; all four listed template files should exist. Check the input's channel identities and calibrated voxel dimensions in Fiji before applying the workflow to a new acquisition type.

---

## 4. Run one sample

Configure Fiji, templates, and Java first (pages 2-3). Then use three arguments separated by spaces, without commas:

```text
bash SCRIPT FILE_PATH CPU_COUNT OUTPUT_FOLDER
```

| Argument | Meaning |
| --- | --- |
| `FILE_PATH` | Path to one input image |
| `CPU_COUNT` | Number of threads; the examples use 8 |
| `OUTPUT_FOLDER` | Parent directory for the sample's results |

Use at least 2 threads: the macro subtracts 1 for preprocessing; CMTK receives the full requested count. Despite `LSF` in the filename, this is a local run and needs no `bsub` command.

### Example: Goro

Change into the package folder once so the command can use the script's filename. The `out` assignment creates a new name for every run; the script creates the output directory.

```bash
cd /Users/otsunah/test/VNC_aligner_local_ver/Necessary_files_VNC_Aligner
out="/Users/otsunah/test/VNC_aligner_local_ver/output_Goro_$(date +%Y%m%d_%H%M%S)"

bash 20x40xVNC_Align_CMTK_localMac_LSF.sh \
    /Users/otsunah/test/VNC_aligner_local_ver/sample/Diego/I3/Oct042026_Goro_VNC_40xoil_Stitch.lsm \
    8 "$out"
```

The backslashes continue one command across lines. You can also put the entire `bash` command on one line. Replace the input path and output-folder name for another sample. DV flipping is already off for all samples.

### Optional console log

To save console output, create the output directory before redirecting the command. Using the same package working directory and a fresh `out` value:

```bash
mkdir -p "$out"
bash 20x40xVNC_Align_CMTK_localMac_LSF.sh \
    /Users/otsunah/test/VNC_aligner_local_ver/sample/Diego/I3/Oct042026_Goro_VNC_40xoil_Stitch.lsm \
    8 "$out" > "$out/run.log" 2>&1
```

Keep Terminal open until alignment finishes. A redirected command is quiet while running. To follow it from another Terminal, use `tail -f /absolute/path/to/output/run.log`. Use Control-C to stop the log viewer without stopping alignment.

Preprocessing details are always in `<output>/<sample-name>/debug/preproc.log`. Confirm `DV flip mode: off`. Check the outputs on page 6 even if the shell reports success: not every Fiji macro error is propagated to the shell.

---

## 5. Run every LSM file in a folder

The script accepts one file per invocation. After the Java setup on page 3, this loop runs every lowercase `.lsm` file directly in `Diego/I3`, one at a time, using 8 threads per sample. DV flipping is already hard-coded off.

```bash
cd /Users/otsunah/test/VNC_aligner_local_ver/Necessary_files_VNC_Aligner
out="/Users/otsunah/test/VNC_aligner_local_ver/output_I3_$(date +%Y%m%d_%H%M%S)"

for file in /Users/otsunah/test/VNC_aligner_local_ver/sample/Diego/I3/*.lsm; do
    [ -f "$file" ] || continue
    bash 20x40xVNC_Align_CMTK_localMac_LSF.sh "$file" 8 "$out"
done
```

The file check skips the literal wildcard if there are no matching files. The loop does not search subfolders. Each sample needs a distinct filename stem because the script uses that stem for its output subfolder.

### Run a different folder

For files directly in `Diego`, change the input pattern to:

```text
/Users/otsunah/test/VNC_aligner_local_ver/sample/Diego/*.lsm
```

For files directly in `AP_inv`, use:

```text
/Users/otsunah/test/VNC_aligner_local_ver/sample/Diego/AP_inv/*.lsm
```

Change the output name to `output_Diego_...` or `output_AP_inv_...` as appropriate. Always set `out` before the loop and use a fresh folder when rerunning with changed settings. No per-sample flip exception or `VNC_DV_FLIP` prefix is needed.

### Progress and failures

Console messages appear in the same Terminal. Every sample gets `debug/preproc.log` in its own output folder; the projection TIFFs share `out/MIP/` with distinct filenames. The simple loop does not create separate `run.log` or `exit_status.txt` files.

The loop waits for each alignment before starting the next and continues after a nonzero shell exit. To stop at the first reported shell failure, replace its `bash` line with:

```bash
    bash 20x40xVNC_Align_CMTK_localMac_LSF.sh "$file" 8 "$out" || break
```

A successful shell exit does not guarantee valid images. Inspect both aligned volumes, both projections, the reference score, and slice direction for each sample. Do not run two jobs for the same sample into the same output folder.

---

## 6. Read the results and choose settings

### Output files

For the two-channel unisex workflow, the main files are:

```text
OUTPUT_ROOT/
  SAMPLE_NAME/
    PRE_PROCESSED_UNISEX_01.nrrd
    PRE_PROCESSED_UNISEX_02.nrrd
    initial.xform
    affine.xform/
    warp.xform/
    SAMPLE_NAME_01.nrrd
    SAMPLE_NAME_02.nrrd
    JRC2018_VNC_UNISEX_447_G15_Score.property
    debug/preproc.log
  MIP/
    SAMPLE_NAME_01.tif
    SAMPLE_NAME_02.tif
```

`FinalOutputs` is no longer created. The sample subfolder holds the aligned data, intermediates, debug logs, and any generated error properties.

Final aligned volumes are `SAMPLE_NAME_01.nrrd` and `_02.nrrd`; the `PRE_PROCESSED` files are intermediate volumes. The macro puts the reference channel in output `_01`. In the validated two-channel runs, input channel 2 became output `_01` and input channel 1 became output `_02`.

With template 1, the verified aligned volumes were **573 x 1119 x 219 voxels**, with voxel spacing approximately **0.461122 x 0.461122 x 0.7 micrometers**. Verify that both volumes open, both projections exist, and the score file contains a number. Inspect reference anatomy against the template; a score alone does not establish anatomical correctness. A score-labelled AVI comparison may also be generated. The active local workflow does not generate a combined V3DRAW file.

### Template and acquisition settings

| Setting | Behavior in this script |
| --- | --- |
| `TEMPNUMBER=1`, `INPUT1_GENDER="u"` | Tested setup: register to 1-micron unisex; resample/score against unisex 447_G15 |
| `TEMPNUMBER=2` | Selects unisex 63x_lowG output template; not tested here |
| `TEMPNUMBER=3` | Female target branch has a filename typo: `RC2018...` must be `JRC2018...` |
| `TEMPNUMBER=4` | Selects male 447_G15 output template; not tested here |
| `INPUT1_GENDER="f"` or `"m"` | Uses initial-template variables not defined in this local script; requires code repair and validation |

Do not assume that changing only template number and gender makes another branch ready to use. Keep `1` and `u` to reproduce the demonstrated workflow.

For 20x images, inspect actual voxel calibration. `objective="40x"` is a label used for metadata/error reporting, not a rescaling control in the active local commands. Changing it to `"20x"` does not configure sampling or template choice. The macro reads image calibration and performs additional geometric processing; validate a representative 20x sample before a batch.

---

## 7. Troubleshooting and safe reruns

| Symptom | What to check |
| --- | --- |
| Preprocess macro could not be found | Set `PREPROCIMG` to the 2026 file in the package root; check `MACRO_DIR`. |
| Fiji cannot be found | Set `FIJI` to the executable inside the chosen application, not its folder. |
| Java startup failure | Verify `JAVA_HOME`, run its `bin/java -version`, and inspect the macro log for actual execution. |
| Unrecognized Fiji command | Install the matching custom plugin and its dependencies, restart Fiji, and check for conflicting copies. |
| CMTK command not found or cannot execute | Check `CMTK`, executable permissions, and compatibility with this Mac. |
| Template cannot be opened | Check external `TempDir` and the selected filename. Template 3 contains a typo in the original code. |
| Initial affine fails | Check preprocessing first: valid `PRE_PROCESSED_UNISEX_01.nrrd` must exist. |
| Wrong reference channel | The 2026 macro forces channel 2 for local `sr`. Changing the shell variable alone does not override this; see page 2. |
| Volumes exist but no projections | Inspect Terminal output or a saved `run.log`; check Template_MIP masks, LUT, and projection plugins. |

### Interrupted runs and cached output

The script skips some stages merely because a path exists. A preprocessing log can be created before preprocessing succeeds. An `initial.xform` can cause both initialization and affine registration to be skipped even if `affine.xform` is missing. The presence of the first final channel can cause reformatting of all channels to be skipped.

After a failed or interrupted run, **use a fresh output directory** and retain the failed folder for diagnosis. Reusing it can silently reuse partial results. Use a new output directory whenever you change the template, reference channel, macro version, or other processing settings.

### Other implementation details

The 2026 macro forces `ShapeAnalysis="false"` internally, even though the shell passes `true`. This differs from the 2022 macro, which forced it on. Local mode also does not perform the same preprocessing-error checks as the non-local branch. Some printed command descriptions differ from the commands actually executed; consult the source when reviewing numerical registration parameters.

Despite `LSF` in its filename, `testmode=1` runs directly on the Mac. This package is not a complete LSF deployment: the other branch depends on missing `common.sh`, cluster parsing, and `/opt/...` software paths. Do not switch to that branch as a way to run a local batch.

### Comparison with earlier results

Historical validation produced different scores across samples and macro versions. Compare the reference anatomy and slice direction against the template for every run, including when the alignment score is high.

These are observed results, not acceptance thresholds. Earlier reports describe the settings at the time of their runs. The current packaged script hard-codes DV flip off and no longer creates `FinalOutputs`. This documentation update did not rerun alignments.
