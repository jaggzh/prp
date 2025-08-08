# PRP (Project Requirements Packager)

> *Finally, a tool that remembers what you installed so you don't have to remember why you installed it.*

**PRP** is a beautiful, minimal tool that tracks your project's build dependencies and creates clean meta-packages for Debian/Ubuntu systems. It's the missing piece between "just `apt install` whatever" and "wait, what did I install for this project again?"

<div align="center">
  <em>Usage overview</em><br>
  <img src="ss/help.png" alt="Usage overview"><br>
</div>

## The Problem

You know the drill:
```bash
$ make
/usr/bin/ld: cannot find -ljpeg: No such file or directory
$ sudo apt install libjpeg-dev
$ make  
/usr/bin/ld: cannot find -lpng: No such file or directory
$ sudo apt install libpng-dev
$ make
/usr/bin/ld: cannot find -ltiff: No such file or directory
# ... 20 minutes later ...
$ make
✓ Success! 
# ... 6 months later ...
$ sudo apt autoremove
# Did I need those 47 -dev packages? 🤷 (Now I have a bunch of
# manually-installed stuff on my system, with no relation to anything else.)
```

**PRP** turns this chaos into clean, tracked, removable dependency management.

## Quick Start

### Workflow 1: Permanent Meta-Package (Recommended)
*Keep the meta-package installed for easy dependency management and reinstallation*

This workflow is most-appropriate when you are definitely going through with the project. The -deps meta-package PRP creates for you, upon removal, frees up the packages from their "manually-installed" state.

```bash
# Download and make executable
curl -O https://raw.githubusercontent.com/yourusername/prp/main/prp
chmod +x prp

# Create a project and track dependencies (auto-installs!)
## This creates the .prp dir local to the current working directory. The
## project name is used for later packaging up as `my-cool-project-deps.deb`
## that depends on the dependencies.
./prp n my-cool-project
./prp t libjpeg-dev libpng-dev libtiff-dev

# Add to project dependencies (respects pre-existing package states)
./prp a

# Create meta-package and install it
./prp install
## Now you have my-cool-project-deps installed, which pulls in all your
## tracked dependencies

# Later: clean removal of everything
./prp uninstall  # Removes meta-package and restores original package states
sudo apt autoremove  # Now safe and predictable
```

### Workflow 2: Temporary Project Cleanup
*Use PRP for intelligent package marking, then remove all tracking*

When you're just fiddling around, and might not even keep this project around. You don't have to go through with the full -deps creation. PRP can track your package installs and then handle the uninstall for you, and you're all clean. **CAVEAT**: Dependencies of packages you installed are NOT tracked by PRP and, unfortunately, when their 'parent' package (the one depending on them, and the one tracked and removed by PRP) -- when that one is removed, Debian will sometimes NOT autoremove them (with `apt autoremove`). You'll find this is the case when a package is installed as a dependency, remains in autoinstalled state, but some other random package had listed it as **Recommended**. Debian does NOT autoremove these!

```bash
# Same setup as above
./prp n temp-experiment
./prp t opencv-dev qt5-dev libboost-dev  # Auto-installs and tracks original states

# Add to dependencies and create meta-package
./prp a
./prp install
# This properly marks packages as auto-installed (unless they were manually installed before)

# Remove the meta-package but keep the intelligent marking
./prp uninstall
# Packages that were NOT manually installed before your project are now marked auto
# Packages that WERE manually installed before remain manual

# Clean up auto-installed packages
sudo apt autoremove
# Only removes packages that were auto-installed and not needed by other manual packages
```

**Key difference:** Workflow 1 keeps the meta-package for ongoing dependency management. Workflow 2 uses PRP to intelligently mark packages for cleanup, then discards the tracking.

## Features That Actually Matter

### 🛡️ **Non-Destructive by Design**
PRP remembers what was manually installed *before* your project started. Your colleague's `libjpeg-dev` won't become auto-removable just because you used it too.

### ⚡ **Track & Install in One Command**
```bash
prp t libjpeg-dev libpng-dev libtiff-dev  # Records state AND installs
# No more manual `sudo apt install` step!
```

### 📦 **One Command, Multiple Packages**
```bash
prp a libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev
# Because copying from Stack Overflow should be this easy
```

### 🎯 **Tentative Tracking**
```bash
prp t opencv-dev qt5-dev  # "I might need these..." (auto-installs)
# (experiment for 3 hours)
prp a                     # "OK, keeping them all"
```

### 🧹 **Clean Meta-Packages**
Creates proper Debian packages that `apt` understands. No weird scripts, no magic files in `/usr/local/`, no "trust me bro" installations.

### 🔄 **Smart State Management**
- **Track** packages and auto-install while preserving original states
- **Add** packages individually or in bulk with intelligent auto-marking
- **Remove** cleanly with restoration to pre-project configuration

## Commands Reference

| Command | What it does | Functional purpose |
|---------|-------------|-------------------|
| `prp n PROJECT` | Create new project (makes .prp/ dir; PROJECT-deps is the future package name) | Initialize dependency tracking in current directory |
| `prp t PKG [PKG2...]` | Track packages & auto-install | Record original state before installation |
| `prp t PKG -I` | Track only (no install) | Record state without changing system |
| `prp a PKG [PKG2...]` | Add packages to dependencies | Mark packages as project requirements |
| `prp a` | Add all tentative packages | Confirm all tracked packages as dependencies |
| `prp e PKG` | Add to tentative (evaluating) | Track package without committing to it |
| `prp r PKG` | Remove from deps & mark for cleanup | Remove package from project requirements |
| `prp s` | Show status | Display current project state and package lists |
| `prp install` | Build & install meta-package | Create system package that depends on all requirements |
| `prp uninstall` | Remove meta-package & cleanup | Remove meta-package and restore original package states |

### Track Command Options

| Flag | Effect | When to use |
|------|--------|----------|
| `prp t PKG` | Track & install (default) | Normal workflow - install package while recording its original state |
| `prp t PKG -I` | Track only, don't install | When you want to record state before deciding whether to install |
| `prp t PKG --no-install` | Same as `-I` | Alternative flag for the same behavior |
| `prp t PKG --update` | Refresh recorded state | When package state changed outside of PRP and you need to update records |

## Real-World Workflows

### Standard Workflow (Recommended)
*Track packages first to record their original state, then add to dependencies*

```bash
prp n mediapipe-experiment
prp t libjpeg-dev libpng-dev libtiff-dev opencv-dev  # Auto-installs & tracks state
prp a  # Adds all tentative, respects pre-project states
prp install
```

### Manual Installation Control
*Track packages without auto-installing, then install manually when ready*

```bash
prp n mediapipe-experiment
prp t libjpeg-dev libpng-dev -I  # Track state but don't install yet
sudo apt install libjpeg-dev    # Install manually when ready
sudo apt install libpng-dev
prp a  # Add all tentative to dependencies
prp install
```

### Skip State Recording  
*Add packages directly without tracking original state (faster but less safe)*

```bash
prp n quick-hack
prp a libjpeg-dev libpng-dev libtiff-dev  # Direct add, no state recording
prp install
```
**Note**: This skips the `prp t` step, so PRP can't restore original package states during cleanup.

### Remove Unwanted Dependencies
*Remove packages from tracking and prepare them for cleanup*

```bash
prp r opencv-dev  # Remove from dependencies, mark for cleanup
prp uninstall     # Remove meta-package and restore original states
sudo apt autoremove  # Clean up auto-marked packages
```

### Incremental Discovery
*Add dependencies as you discover build failures*

```bash
prp n complex-project
prp t libjpeg-dev      # Start with what you know you need
# (build fails: missing libpng)
prp t libpng-dev       # Add newly discovered dependency
# (build fails: missing libtiff)  
prp t libtiff-dev      # Keep adding as needed...
prp a                  # Finalize all tentative packages
prp install
```

## Installation

### Requirements
- Python 3.6+ 
- `equivs` package (`sudo apt install equivs`)
- `python3-yaml` (`sudo apt install python3-yaml`)

### Install PRP
```bash
# Method 1: Direct download
curl -O https://raw.githubusercontent.com/yourusername/prp/main/prp
chmod +x prp
sudo mv prp /usr/local/bin/

# Method 2: Clone and symlink
git clone https://github.com/yourusername/prp.git
sudo ln -s $(pwd)/prp/prp /usr/local/bin/prp
```

## Limitations & Known Issues

### Dependency Tracking Scope
**PRP only tracks packages you explicitly tell it to track.** When you `prp t libopencv-dev`, PRP doesn't monitor the 15 dependencies that `apt` pulls in automatically (like `libopencv-core`, `libpng16-16`, etc.).

This is usually fine because:
- Those dependencies get marked as auto-installed by `apt`
- They'll be removed by `apt autoremove` when no longer needed

**However**, there's a Debian quirk: packages that are *recommended* by other installed packages won't be auto-removed, even if marked as auto-installed. So if you have Package A installed that recommends `libfoo-dev`, and you also install `libfoo-dev` as a dependency of your tracked package, `apt autoremove` won't remove `libfoo-dev` even after you uninstall your project.

**Workaround**: If you notice packages lingering after cleanup, you can manually `sudo apt remove` them or use `apt-mark showmanual` to review what's marked as manually installed.

### Cross-Project Package Sharing
If you use the same package across multiple PRP projects, the "original state" recorded by the first project takes precedence. This is generally correct behavior, but can be confusing if you change a package's manual/auto status outside of PRP.

## FAQ

**Q: Yet another package manager?**  
A: No. PRP is a *dependency tracker* that creates standard Debian packages. It's `equivs` with a brain and a memory.

**Q: Why does `prp t` install packages automatically now?**  
A: Because 90% of the time, you track packages because you're about to install them. Use `-I` when you want to track without installing.

**Q: What about Docker/containers?**  
A: Great for production. This is for development where you need to `apt install` things while figuring out what you actually need.

**Q: Does it work on RPM systems?**  
A: Not yet. PRs welcome for the brave souls who want to implement `zypper`/`dnf` support.

**Q: My company has security policies about meta-packages...**  
A: PRP generates standard Debian control files that your security team can audit. The generated `.deb` files are just dependency lists—no code execution.

**Q: What if I already have a mess of manually installed packages?**  
A: `prp t package1 package2 --update` will record current states. PRP is designed to be adopted gradually.

**Q: Can I track packages that aren't in the main repos?**  
A: Yes! Use `prp a package-name -f` to force-add packages from PPAs or custom repos. `prp t` will skip installation for packages it can't find but still track them.

**Q: Should I use Workflow 1 or 2?**  
A: Use Workflow 1 for ongoing projects where you want easy dependency reinstallation later. Use Workflow 2 for temporary experiments where you just want intelligent cleanup without keeping tracking files.

## Under the Hood

PRP stores project state in `.prp/state.yaml`:

```yaml
project:
  name: my-project
  package_name: my-project-deps # This becomes the .deb filename
dependencies:
  - libjpeg-dev
  - libpng-dev
tentative:
  - opencv-dev
original_state:
  libjpeg-dev:
    was_installed: true
    was_manual: true  # Won't mark as auto on cleanup
  libpng-dev:
    was_installed: false
    was_manual: false  # Was installed by this project
```

The generated meta-package is just a standard Debian package that depends on your tracked packages. No magic, no lock-in.

When you `prp t package-name`, PRP:
1. Records the package's current state (installed/not installed, manual/auto)
2. Installs the package via `sudo apt install -y` (unless `-I` flag used)
3. Adds to tentative list for later confirmation with `prp a`

## Contributing

Found a bug? Have a feature idea? The codebase is refreshingly simple:

- `core/` - Project state management
- `platform/` - Debian/Ubuntu integration  
- `hooks/` - Extension points (future)

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License. Because dependency management pain should be freely shared.

---

*PRP: For developers who've learned that "I'll remember what I installed" is the first lie we tell ourselves.*
