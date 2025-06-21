# 🚀 Amber Installation & Setup on WSL2 (Ubuntu) - Complete Tutorial

This guide walks you step-by-step through setting up **Amber** on **WSL2** with both **GPU** and **CPU support**, including backup & git integration.

---

## ✅ STEP 1 – Installing WSL2 & Ubuntu

1. **Open PowerShell as Administrator**  
   Search for `PowerShell`, right-click → Run as Administrator.

2. **Run the following commands one by one:**
   ```powershell
   wsl --install
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   wsl --set-default-version 2
   ```

3. **List the available Ubuntu versions to install:**
   ```powershell
   wsl --list --online
   ```
   Select the latest version from the list, for example:
   ```powershell
   wsl --install -d Ubuntu-22.04
   ```

   Once installation completes, you’ll see:
   ```
   Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.167.4-microsoft-standard-WSL2 x86_64)
   ```

4. **Create a user and password** when prompted. Save them so you don’t forget.

5. You can now search for "Ubuntu" from the start menu and open the terminal.
   You may be prompted to create another user—using the same values is fine.

> 🧠 **Note**: When entering passwords in the terminal, nothing will show (not even asterisks). Just type your password and press Enter.

---

## 🔧 STEP 2 – Updating Ubuntu & Installing Essentials

Once inside Ubuntu:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🛠 STEP 3 – Install CMake & Access Explorer

1. To open your Ubuntu folder in Windows Explorer:
   ```bash
   explorer.exe `wslpath -w "$PWD"`
   ```

2. **Install CMake**  
   Download the `.sh` installer from [CMake Downloads](https://cmake.org/download/) (x86_64 Linux version).  
   Move it to your Ubuntu folder and run:
   ```bash
   sh cmake-<version>-linux-x86_64.sh
   ```
   also run
   ```bash
   sudo apt install cmake
   ```
   Follow the prompts and accept the license with `y`.

---

## 🐍 STEP 4 – Install Miniforge (Minimal Conda)

1. Download from: [Miniforge Releases](https://github.com/conda-forge/miniforge/releases)
2. Move the `.sh` file to your Ubuntu folder.
3. Run:
   ```bash
   sh Miniforge3-*.sh
   ```
   example:
   ```bash
   sh Miniforge3-25.3.0-3-Linux-x86_64.sh
   ```

5. Accept all prompts (press Enter or type `yes` when prompted).

6. **Close and reopen** your Ubuntu terminal to apply changes.

---

## 🧪 STEP 5 – Prepare Amber Tools

### 5.1 – Create & Activate Conda Environment

```bash
conda create --name AmberTools25 python=3.12
conda activate AmberTools25
```

### 5.2 – Install AmberTools

```bash
conda install dacase::ambertools-dac=25
```

### 5.3 – Download Amber24

Visit: [https://ambermd.org/GetAmber.php](https://ambermd.org/GetAmber.php)  
Download `pmemd24.tar.bz2` for non-commercial use. Move it to your Ubuntu folder.

### 5.4 – Extract Amber24

```bash
conda deactivate
tar xvfj pmemd24.tar.bz2
```

---

## ⚡ STEP 6 – GPU & CPU Setup (Optional)

### 6.1 – Enable GPU Support (CUDA 11.6.2)

1. **Install CUDA 11.6.2 Toolkit (without overwriting your driver):**

   ```bash
   wget https://developer.download.nvidia.com/compute/cuda/11.6.2/local_installers/cuda_11.6.2_510.47.03_linux.run
   chmod +x cuda_11.6.2_510.47.03_linux.run
   sudo sh cuda_11.6.2_510.47.03_linux.run --silent --toolkit --override
   ```

   > ⚠️ Do **not** install the driver when prompted — your system or WSL already provides a working driver via Windows.

2. **Add CUDA 11.6 to your environment variables** by adding the following lines to your `~/.bashrc` file:

   ```bash
   export CUDA_HOME=/usr/local/cuda-11.6
   export PATH=$CUDA_HOME/bin:$PATH
   export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
   ```

   Then apply changes:

   ```bash
   source ~/.bashrc
   ```

3. **Verify CUDA installation:**

   ```bash
   nvcc --version
   ```

   Expected output should show something like:
   ```
   Cuda compilation tools, release 11.6, V11.6.x
   ```

4. Verify with:
   ```bash
   nvidia-smi
   ```

5. Edit `pmemd24_src/build/run_cmake` and **change all** `DCUDA=FALSE` to `DCUDA=TRUE`.

### 6.2 – Enable CPU (MPI) Support

```bash
sudo apt install -y openmpi-bin libopenmpi-dev
```

Then edit `run_cmake` again and **change all** `DMPI=FALSE` to `DMPI=TRUE`.

---

## 🔨 STEP 7 – Final Installation

Install required packages:

```bash
sudo apt install flex bison gcc-10 g++-10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 100
```

Compile Amber:

```bash
cd pmemd24_src/build
./run_cmake
make install -j$(nproc)
```

> ⚠️ **Note**: This may take a while and will use 100% of your CPU. If it completes too quickly or shows errors, investigate further.
> Final result should look like this
> ![final_result](https://github.com/user-attachments/assets/4286dab5-f13c-40e6-b9a1-294d9627476e)


---

## ✅ STEP 8 – Test Installation

1. Activate Amber environment:
   ```bash
   source /home/asrakan/miniforge3/envs/AmberTools25/amber.sh
   echo "source /home/asrakan/miniforge3/envs/AmberTools25/amber.sh" >> ~/.bashrc
   source ~/.bashrc
   ```

2. Run the CUDA test:
   ```bash
   cd $AMBERHOME
   make test.cuda.serial
   ```
3. Run CUDA pmemd test
 ```bash
   source ~/pmemd24/amber.sh
   which pmemd.cuda
   pmemd.cuda -h
```

---


## 💾 STEP 9 – Backup Your WSL

In PowerShell:

```powershell
New-Item -Path "C:\WSL_Backups" -ItemType Directory
wsl --export Ubuntu C:\WSL_Backups\Ubuntu_Backup.tar
```

To restore:
```powershell
wsl --unregister Ubuntu
wsl --import Ubuntu C:\WSL\UbuntuRestored C:\WSL_Backups\Ubuntu_Backup.tar
```

To keep the old version and create a new one:
```powershell
wsl --import UbuntuBackup C:\WSL\UbuntuBackup C:\WSL_Backups\Ubuntu_Backup.tar
```

---

## 🔄 STEP 10 – Git Integration (Optional)

1. Download [GitHub Desktop](https://desktop.github.com/download/)
2. Install and create a GitHub account.
3. Create a folder in your Ubuntu Explorer directory.
4. Add this as a **local repo** and optionally publish it to GitHub.

