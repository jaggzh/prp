# PRP (Project Requirements Packager)

> *Finally, a tool that remembers what you installed so you don't have to remember why you installed it.*

**PRP** is a beautiful, minimal tool that tracks your project's build dependencies and creates clean meta-packages for Debian/Ubuntu systems. It's the missing piece between "just `apt install` whatever" and "wait, what did I install for this project again?"

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
# Did I need those 47 -dev packages? 🤷
```

**PRP** turns this chaos into clean, tracked, removable dependency management.

## Quick Start

```bash
# Download and make executable
curl -O https://raw.githubusercontent.com/yourusername/prp/main/prp
chmod +x prp

# Create a project
./prp n my-cool-project

# Track what you're about to install (safe mode)
./prp t libjpeg-dev libpng-dev libtiff-dev

# Install manually to test
sudo apt install libjpeg-dev libpng-dev libtiff-dev

# Add to project (respects pre-existing package states)
./prp a

# Create meta-package and install
./prp install

# Later: clean removal
./prp uninstall
sudo apt autoremove  # Now safe and predictable
```

## Features That Actually Matter

### 🛡️ **Non-Destructive by Design**
PRP remembers what was manually installed *before* your project started. Your colleague's `libjpeg-dev` won't become auto-removable just because you used it too.

### 📦 **One Command, Multiple Packages**
```bash
prp a libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev
# Because copying from Stack Overflow should be this easy
```

### 🎯 **Tentative Tracking**
```bash
prp t opencv-dev qt5-dev  # "I might need these..."
# (experiment for 3 hours)
prp a                     # "OK, keeping them all"
```

### 🧹 **Clean Meta-Packages**
Creates proper Debian packages that `apt` understands. No weird scripts, no magic files in `/usr/local/`, no "trust me bro" installations.

### 🔄 **Smart State Management**
- **Track** packages before installing to preserve original states
- **Add** packages individually or in bulk with intelligent auto-marking
- **Remove** cleanly with restoration to pre-project configuration

## Commands Reference

| Command | What it does | Why you'll love it |
|---------|-------------|-------------------|
| `prp n PROJECT` | Create new project | Because naming things is hard enough |
| `prp t PKG [PKG2...]` | Track packages (record current state) | Paranoia that pays off |
| `prp a PKG [PKG2...]` | Add packages to dependencies | The meat and potatoes |
| `prp a` | Add all tentative packages | For the "yep, all of them" moment |
| `prp e PKG` | Add to tentative (evaluating) | "I think I need this but..." |
| `prp r PKG` | Remove from deps & mark for cleanup | Mistakes were made |
| `prp s` | Show status | Your project at a glance |
| `prp install` | Build & install meta-package | Make it official |
| `prp uninstall` | Remove meta-package & cleanup | The great cleanup |

## Real-World Workflows

### The Paranoid Workflow (Recommended)
*For when you've been burned before*

```bash
prp n mediapipe-experiment
prp t libjpeg-dev libpng-dev libtiff-dev opencv-dev
sudo apt install libjpeg-dev libpng-dev libtiff-dev opencv-dev
prp a  # Adds all tentative, respects pre-project states
prp install
```

### The YOLO Workflow  
*For when you're feeling lucky*

```bash
prp n quick-hack
prp a libjpeg-dev libpng-dev libtiff-dev
prp install
```

### The "Oh No" Workflow
*For when things went sideways*

```bash
prp r opencv-dev  # Oops, didn't need this
prp uninstall     # Nuclear option: remove everything
sudo apt autoremove  # Now safe
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

## FAQ

**Q: Yet another package manager?**  
A: No. PRP is a *dependency tracker* that creates standard Debian packages. It's `equivs` with a brain and a memory.

**Q: What about Docker/containers?**  
A: Great for production. This is for development where you need to `apt install` things while figuring out what you actually need.

**Q: Does it work on RPM systems?**  
A: Not yet. PRs welcome for the brave souls who want to implement `zypper`/`dnf` support.

**Q: My company has security policies about meta-packages...**  
A: PRP generates standard Debian control files that your security team can audit. The generated `.deb` files are just dependency lists—no code execution.

**Q: What if I already have a mess of manually installed packages?**  
A: `prp t package1 package2 --update` will record current states. PRP is designed to be adopted gradually.

## Under the Hood

PRP stores project state in `.prp/state.yaml`:

```yaml
project:
  name: my-project
  package_name: my-project-deps
dependencies:
  - libjpeg-dev
  - libpng-dev
tentative:
  - opencv-dev
original_state:
  libjpeg-dev:
    was_installed: true
    was_manual: true  # Won't mark as auto on cleanup
```

The generated meta-package is just a standard Debian package that depends on your tracked packages. No magic, no lock-in.

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