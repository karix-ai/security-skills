---
name: git-identity-hygiene
description: Keep personal data out of a public repo — commit emails, real names, photo GPS metadata, document metadata, internal hostnames. Use before open-sourcing, when the user wants to stay pseudonymous, or asks "what does my repo reveal about me".
---

# Git identity hygiene

Every commit carries an author name and email, forever, readable by anyone with
`git log`. Every phone photo can carry GPS coordinates. Every exported PDF or DOCX can carry
the author's full name and the path on their laptop. None of it shows on the GitHub page,
which is why people forget it is there.

## 1. Commit emails

See what the repo already reveals:

```bash
git log --all --format='%an <%ae>' | sort -u
```

Use the GitHub noreply address from now on (Settings → Emails shows it, format `ID+username@users.noreply.github.com`):

```bash
git config --global user.email "ID+username@users.noreply.github.com"
git config --global user.name  "username"
```

On GitHub: Settings → Emails → turn on **Keep my email addresses private** and **Block command line pushes that expose my email**.

To rewrite existing commits (only if the repo is not widely cloned yet; this changes every hash):

```bash
# mailmap.txt:  New Name <ID+username@users.noreply.github.com> <personal@gmail.com>
git filter-repo --mailmap mailmap.txt
git push --force --all && git push --force --tags
```

## 2. Image and document metadata

```bash
# Find images with GPS data
exiftool -r -if '$GPSLatitude' -p '$FileName $GPSLatitude $GPSLongitude' .

# Strip all metadata from images in place
exiftool -r -all= -overwrite_original -ext jpg -ext jpeg -ext png -ext heic .

# Check PDFs and Office files for author/creator
exiftool -r -Author -Creator -LastModifiedBy -ext pdf -ext docx -ext xlsx .
```

Screenshots also leak: browser tabs, bookmarks bar, username in a terminal prompt, desktop file names, notification previews. Crop before committing.

## 3. Text that identifies you

Search code, comments, tests and docs:

```bash
git grep -n -I -i -E "(/Users/|/home/|C:\\\\Users\\\\)"     # local paths with your username
git grep -n -I -E "[0-9]{3}[- ]?[0-9]{3}[- ]?[0-9]{2,4}"    # phone-like numbers (noisy, read the hits)
git grep -n -I -i -E "(localhost:|192\.168\.|10\.[0-9]+\.|\.internal|\.local)"
```

Also: real names in test fixtures, your company's internal URLs, customer data in seed files, your city in a timezone or weather example.

## 4. Account-level

- Profile: no phone, no home address, location at country level or blank.
- Profile photo: not one reused from personal social media (reverse image search links accounts).
- Don't link personal social accounts you want kept separate.
- Turn on 2FA (authenticator app, not SMS) and save recovery codes offline.

## Procedure

1. List author identities in history. Flag anything that is not a noreply or deliberately public address.
2. Scan images and documents for metadata; strip it.
3. Grep for local paths, internal hosts and personal identifiers; read every hit.
4. Fix git config for future commits; rewrite history only if the user agrees and understands hashes will change.
5. Review the public profile fields.

## Output

What was found (by category), what was fixed, the commands run, and anything the user must change in the GitHub UI.
