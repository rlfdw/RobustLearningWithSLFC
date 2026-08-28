# Anonymous project page — setup

Static site, no build step, no external dependencies except web fonts.
Everything identifying has been removed; the checklist below is what you still need to do.

```
index.html            the page
figures/              your figure exports (currently empty)
media/                the demo video (currently empty)
paper.pdf             the anonymised PDF (add this)
```

Until you add a figure, its slot renders as a dashed box naming the file it expects. Nothing breaks.

---

## 1. Add your assets

Re-export every figure from source rather than downloading it from the old site — the
Google-hosted copies are served under your site's ID and carry whatever metadata your
plotting library wrote in.

```
figures/architecture.png
figures/franka-kitchen.png
figures/robot-writing.png
figures/fetchpush.png
figures/frechet.png
figures/video-poster.png     optional
media/robot-writing.mp4
paper.pdf
```

Then strip metadata from all of it:

```bash
exiftool -all= -overwrite_original figures/*.png
ffmpeg -i media/robot-writing.mp4 -map_metadata -1 -c copy media/clean.mp4 && mv media/clean.mp4 media/robot-writing.mp4
qpdf --linearize paper.pdf paper-clean.pdf && exiftool -all= -overwrite_original paper-clean.pdf && mv paper-clean.pdf paper.pdf
```

PDFs are the worst offender here: LaTeX writes the author field, and `\pdfinfo` or hyperref
metadata survives compilation even when the title page is anonymised.

Watch the video frame by frame before you commit it. The things that catch people out are
lab signage, name badges, whiteboards, reflections in the robot's housing, and a visible
terminal showing a username or an absolute path.

## 2. Add the code link

Mirror your repo through <https://anonymous.4open.science>, then replace the placeholder URL
in `index.html` and remove `aria-disabled="true"` from that link so it becomes clickable.

Before mirroring, grep the repo for the obvious strings — institution name, author names,
email addresses, `/home/<username>/` paths in configs, W&B entity names, and cluster
hostnames in job scripts. The `.git` history counts too: commit author names and emails
travel with the repo, so mirror from a squashed export rather than the working repo if the
history is dirty.

## 3. Publish

Create a **new** GitHub account used only for this submission — not your usual one, since
repo ownership, stars and follower graph are all public.

```bash
git init && git add . && git commit -m "Project page"
git branch -M main
git remote add origin https://github.com/<throwaway>/<repo>.git
git push -u origin main
```

Then Settings → Pages → deploy from `main` / root. The page ships with
`<meta name="robots" content="noindex, nofollow">`, so it will not be indexed.

Set `git config user.name` and `user.email` **inside this repo** before the first commit, or
your real address ends up in the commit objects permanently.

## 4. Final pass

Open the deployed URL in a private window, logged out of everything, and check:

- [ ] View source, search for: your surname, co-authors' surnames, `monash`, `arxiv`, your email prefix
- [ ] Same search inside every figure — axis labels, legends, watermarks, embedded logos
- [ ] `exiftool figures/* media/* paper.pdf` returns no Author, Creator or Producer fields
- [ ] Video contains no lab-identifying frames
- [ ] The old Google Site is untouched and still live at its own URL
- [ ] Repo has no `.git` history carrying your identity
- [ ] Your venue permits a preprint of this work to exist publicly — check the specific policy

## Notes

The hero animation is a schematic, not measured data, and is captioned as such. If you would
rather not have an illustration on a review page at all, delete the `<div class="signature">`
block and the CSS between the `signature figure` and `caption` comments.

Colours: the three skill segments use `--k1`, `--k2`, `--k3`. If your paper's figures already
use a fixed colour per skill, change those three variables at the top of `index.html` to match
so the page and the figures agree.
