# README: Hybrid Video/Input Architecture Setup

This guide details the automated installation for routing network input devices (keyboard and mouse) from a terminal to a server while rendering graphics locally to a GPU connected to a physical monitor. This setup targets **Plan 9 from Bell Labs (Legacy x86)** installations utilizing Fossil/Venti storage backends.

---

## Automated Functions

1. **Privilege Elevation:** Connects to the file server console (`/srv/fscons`) to add the active user to the `sys` and `adm` administrative groups if the current session is not authenticated as `eve`.
2. **Timing Validation:** Performs a dry run of `aux/vga` against `/lib/vgadb` to confirm that the `1024x768x8` resolution stanza exists and is valid before modifying files.
3. **Authentication Injection:** Prompts for domain credentials and writes them to active `factotum` memory segments and persistent startup scripts.
4. **Boot Configuration Updates:** Mounts the `9fat` partition and appends system flags (`service=cpu`, `mouseport=ps2`, etc.) to `plan9.ini`.
5. **Network Tuning:** Configures `/net/tcp/ctl` to set a 10-second TCP keep-alive probe to quickly reclaim dropped connections.
6. **Kernel Source & Build Patching:** Modifies `/sys/src/9/pc/9pccpu` to include the VGA driver and updates `/sys/src/9/pc/mkfile` to compile and link `devvga.o`.
7. **Kernel Compilation:** Builds the binary using `mk` and installs it to `/9pccpu` and the `9fat` partition.
8. **Utility Deployment:** Installs a clean-up handler (`/bin/vgareset`) and a verification suite (`/bin/vgatest`).
9. **Profile Integration:** Updates `/usr/$user/lib/profile` to intercept incoming `cpu` sessions, initialize local graphics hardware, bind network inputs, and launch `rio`.
10. **Storage Block Syncing:** Invokes `fshalt -r` to flush the transactional memory caches of the Fossil filesystem, write snapshot milestones to disk sectors, and trigger a hardware warm reboot.

---

## Installation Steps

### 1. Save the Script
Save the installation script on the server filesystem at `/tmp/setup_hybrid.rc`. 

### 2. Run the Script
Execute the script using the `rc(1)` shell as user `glenda`:
```rc
rc /tmp/setup_hybrid.rc
```

### 3. Enter Authentication Parameters
Provide the required parameters when prompted:
1. **Network Domain:** The authentication domain (e.g., `choice.com`).
2. **User Name:** The active user account handle (e.g., `glenda`).
3. **Secret Key:** The account password.

### 4. Sync NVRAM Key
When the script calls `auth/wrkey` at the end of the run, follow the prompts to manually commit your host key configuration down into the physical NVRAM partition.

---

## Verification

The server will automatically reboot after the file cache flush finishes. To verify the configuration from your remote seat:

1. Go to the remote terminal.
2. Connect to the server:
   ```rc
   cpu -h server_name
   ```
3. After `factotum` authenticates, `rio` will automatically launch on the server's physical display via the long cable link.
4. Run the verification suite within the graphical workspace to confirm the integrity of the namespace files:
   ```rc
   /bin/vgatest
   ```
