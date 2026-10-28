# Assignment 2 Report

- **Name**: TODO
- **Student ID**: TODO
- **Email**: TODO
- **Group**: TODO [BBADBA 5A | BBADBA 5B | PPLEDBA 5A | BDBA 3A]

> Sections marked **[from results]** are filled in from `artifacts/RESULTS.md`,
> which `main.py train` writes, once the real training run is done.

## Business problem, and the cost of being wrong

**The decision.** An electronics contract manufacturer solders through-hole
and leaded components onto control boards. After soldering, a camera
station takes about 40 close-up images ("windows") of each board's solder
joints. Today an operator looks at the flagged windows and decides, for each
one: **pass** (the board moves on to functional test and shipping) or **fail**
(the board goes to the rework bench, where a technician re-solders the joint).
The model makes that call per window. A window is positive when it shows at
least one defective joint: a bridge between pins (a short), a detached or
insufficient joint (an open, or one that cracks later), or a double joint.

**Who acts on it.** The model's output routes the board. "Fail" sends it to a
rework technician, and "pass" sends it down the line. The third option, "review",
puts the window on an inspector's screen for a few seconds before routing.

**What each mistake costs.**

- **False negative: a defective joint is passed (≈ €72).** The defect
  doesn't disappear. It is found later, where finding it costs more. That is
  the "rule of ten": each step a defect travels down the chain multiplies the
  cost of dealing with it ([NXP](https://www.nxp.com/company/about-nxp/smarter-world-blog/BL-ECONOMICS-TEST-YIELD-COST-LATE-DETECTION)
  puts the jump at roughly an order of magnitude per stage). Two outcomes are
  assumed:
  - **Caught at functional test (80%):** the board fails, a technician
    debugs it (≈15 min at €40/h ≈ €10), and it is reworked and re-tested
    (≈ €5). Total ≈ **€15**.
  - **Missed until the field (20%):** intermittent joints often pass a powered
    test and crack under thermal cycling. That means a customer return, a
    replacement board, shipping and failure analysis. Total ≈ **€300**.
  - **Expected cost:** 0.8 × 15 + 0.2 × 300 = **€72**.
- **False positive: a good joint is failed (≈ €1.50).** A technician picks
  the board up, inspects and touches up a joint that was fine: about 2 minutes
  at €45/h fully loaded, ≈ €1.50. No product is lost.
- **True positive (€1.50).** The same rework, now worth doing, because it
  prevents a €72 escape.
- **True negative (€0).**
- **Review (€0.12 per window).** An inspector looks at one window on screen:
  ≈12 s at €35/h. Reviewers are assumed good but not perfect: they catch 95%
  of defects and wrongly fail 2% of good windows.

**The asymmetry.** A missed defect costs about **48 times** a false alarm
(€72 vs €1.50). That alone would push the threshold low, so that we fail
anything suspicious. But defects are rare. About **0.2% of windows** contain one
(see the base rate below), so false alarms come from the other 99.8% of
windows. The two effects pull in opposite directions. That is why the
threshold is chosen by sweeping, not by feel: with 1 defect per 500 windows,
failing too eagerly costs `499 × €1.50` of rework to save one €72 escape.

**Base rate.** Wave and selective soldering of through-hole boards typically
yields first-pass rates of 90–95% per board. With ~40 windows per board, a
board defect rate of ~8% corresponds to roughly **0.2% of windows**
(1 − (1 − 0.002)^40 ≈ 7.7%). The test split is far richer in defects than
that, because the dataset was collected to show defects. So every cost
in this report is re-weighted from the test set's error rates to this
production base rate, rather than read off the raw test counts.

**What this means for the rest of the report.** The metric that matters is
**image-level recall on defects at a low false-alarm rate**, not mAP. A
model that draws perfect boxes but misses one bridge in a hundred is worse
for this business than a sloppier one that never misses. The "Good" class is
useful to the model (it learns what normal looks like), but it carries no
decision weight of its own.

## Dataset

- **Workspace / project / version:** `work-6qkmv` / `solder-2-hcmiq` / **v37**
- **Link:** https://universe.roboflow.com/work-6qkmv/solder-2-hcmiq/dataset/37
- **License:** CC BY 4.0. Used as published, not forked.
- **Task:** object detection, 5 classes: `Good`, `Bridge`, `Detached`,
  `Double`, `Insufficient`.
- **One image:** a greyscale, top-down close-up of two or three soldered
  leads, like the frame an inspection camera captures. Each joint is boxed and
  labelled, so images with only `Good` boxes are true negatives for the
  business decision.

**Why it serves the problem rather than resembling it.** It is the same
viewpoint and lighting as an inline inspection camera. It also labels good
joints explicitly. That matters: the most popular PCB-defect sets on Universe
(the bare-board "missing hole / mouse bite / spur" family) contain a defect in
every single image, so they can't teach the model what a passable board looks
like. Without negatives there is no false-positive rate to put a cost on.

**Splits.** Version 37 ships 4,239 train and 608 validation images and **no
test split**. Roboflow applies its augmentation (×3 outputs: flips, 90°
rotations, crop, shear, brightness, noise) only to the training split, so:

- Roboflow's **validation split (608 images, unaugmented) is used as the
  held-out test set**. It is never seen during training or model selection.
- A **validation split for early stopping** is carved out of Roboflow's
  training split: 10% of *source images*, keeping all three augmented copies
  of an image on the same side. Splitting by file instead would put
  near-duplicates in both train and validation, and the validation score would
  overstate the model.

## What the dataset's composition implies

**[from results]** Paste the class / split / object-size table from
`artifacts/RESULTS.md`, then interpret it:

- Which defect classes have too few test instances (under ~50) for their
  per-class precision and recall to be trusted.
- The share of test images that contain a defect, compared with the 0.2%
  production base rate. This is why costs are re-weighted.
- Object size relative to the image. Joints are large (a big share of a
  640×640 window), so small-object failure is not the risk here.
- The test images come from the same camera and boards as training. The test
  number therefore says nothing about a new board design, a new camera or new
  lighting.

## Finetuning and baselines

**[from results]**

## Per-class evaluation and failure modes

**[from results]**

## Cost matrix and operating point

**[from results]**

## What a person still has to check

**[from results]**

## Limitations & next steps

**[from results]**

## Generative AI use disclosure

TODO: state what you used and how. For example: "I used Claude (Anthropic) to help
draft the pipeline code and report structure. I chose the problem, checked
every number against my own training run, and can explain each decision."
