# Git LFS Setup & Troubleshooting

## The Problem

```
remote: error: File kompress-v2-base/models/onnx/kompress-int8-wo.onnx is 261.35 MB; 
             this exceeds GitHub's file size limit of 100.00 MB
```

This error occurs even when git-lfs is installed if files are committed before being added to LFS tracking.

---

## Sender Side (Push / Contribute)

### 1. Install Git LFS

```bash
git lfs install
```

### 2. Track the file BEFORE committing

```bash
git lfs track "*.onnx"
git add .gitattributes
git add path/to/large-file.onnx
git commit -m "Add large file"
```

> **Important:** `git lfs track` only affects future commits. Files already committed as raw content must be migrated.

### 3. Fix an Already-Committed File (Migrate to LFS)

If a large file is already committed without LFS tracking, use `git lfs migrate` to retroactively convert it:

```bash
# Preview what will be migrated (files above 50MB)
git lfs migrate info --everything --above 50mb

# Convert all large files to LFS tracking
git lfs migrate import --above 50mb --everything

# Push the changes
git push origin <branch>
```

### 4. Verify LFS Tracking (Sender)

```bash
# Should list tracked files
git lfs ls-files

# If empty, the file is NOT in LFS
```

### 5. Check the File Content

```bash
# LFS pointer (correct — small text file)
git show HEAD:path/to/file.onnx

# Raw content (wrong — large binary committed directly)
# Shows binary data instead of pointer
```

---

## Receiver Side (Pull / Clone)

LFS is **transparent** on the receiving side — no special commands needed after `git lfs install`.

### 1. One-time setup per machine

```bash
git lfs install
```

### 2. Clone or Pull — works like normal git

```bash
# Clone a repo with LFS files
git clone git@github.com:rsrini7/Temp.git
# LFS files are automatically downloaded after clone

# Or for an existing repo
git pull
# LFS content is fetched automatically
```

### 3. Verify LFS Content Downloaded Correctly

```bash
# Confirm LFS is tracking the file
git lfs ls-files
# Should list the .onnx file

# Check the file is actual content (not a pointer)
git show HEAD:kompress-v2-base/models/onnx/kompress-int8-wo.onnx | head -3
# Should show binary content, not a text SHA pointer
```

### 4. If LFS Content Doesn't Download Automatically

If `git lfs ls-files` is empty after a pull, manually fetch:

```bash
git lfs pull
# or
git lfs fetch --all
git lfs checkout
```

### 5. Fresh Clone vs Existing Clone

| Scenario                    | What to do                                |
|-----------------------------|-------------------------------------------|
| Fresh clone                 | Nothing extra — LFS works out of the box  |
| Existing clone (up-to-date) | `git lfs pull` if files show as pointers  |
| New machine                 | `git lfs install` first, then pull/clone  |

---

## LFS File Size Limits

| Service     | Max File Size |
|-------------|---------------|
| GitHub Free | 100 MB        |
| GitHub Pro  | 100 MB        |
| GitHub Team | 100 MB        |
| GitHub Ent  | 5 GB          |

## Best Practices

1. **Always `git lfs track` before adding large files** — add the pattern to `.gitattributes` first
2. **Never use `git add -A`** — stage files explicitly to avoid accidentally committing large files without LFS
3. **Migrate early** — the larger the repo history, the longer `migrate` takes
4. **Use `.gitattributes`** — commit this file so collaborators also use LFS for matching files

## Common Commands

```bash
# Sender side
git lfs install              # Initialize LFS in a repo
git lfs track "*.psd"        # Track all .psd files
git lfs track "*.onnx"       # Track all .onnx files
git lfs ls-files             # List files currently tracked by LFS
git lfs migrate info         # Show LFS migration info
git lfs migrate import       # Migrate history to LFS

# Receiver side
git lfs pull                 # Pull LFS objects (if auto-fetch failed)
git lfs fetch                # Fetch LFS objects
git lfs checkout             # Checkout LFS files to working directory
```

## How LFS Works (Summary)

1. **Sender**: Large file is stored in LFS storage (separate from git). Git commits a small pointer file instead.
2. **Push**: Pointer goes to git remote, actual file goes to LFS remote.
3. **Pull/Clone**: Git fetches the pointer, LFS fetches the actual file and replaces the pointer locally.
4. The `.gitattributes` file (committed to git) tells git which files are LFS-tracked.