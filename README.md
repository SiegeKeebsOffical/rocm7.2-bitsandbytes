# rocm7.2-bitsandbytes
compiled version of bitsandbytes that works with rocm 7.2

I have no idea what I'm doing and I just had gemini walk me through it with constant troubleshooting... this was specifically compiled for the gfx1151 in the AMD Strix Halo 395+ AI chip

## **ROUGH STEPS:**

sudo pacman -S python python-pip

### Create the venv without pip (this will succeed)
python3.12 -m venv ~/bnb_test/test_env --without-pip

### Activate it (assuming Fish shell)
source ~/bnb_test/test_env/bin/activate.fish

### Manually install pip into this sandbox
curl https://bootstrap.pypa.io/get-pip.py | python


### Before we compile, we need that gfx number. CachyOS usually puts ROCm in /opt/rocm/.
/opt/rocm/bin/rocminfo | grep gfx


### Clone and enter
git clone https://github.com/bitsandbytes-foundation/bitsandbytes.git
cd bitsandbytes

### Install build dependencies
pip install setuptools wheel thop tqdm
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm7.2 --no-cache-dir

python -c "import torch; print(f'ROCm available: {torch.cuda.is_available()}'); print(f'Current device: {torch.cuda.get_device_name(0)}')"

sudo pacman -S hipsparse hipblas hiprand hipsolver rocsparse rocblas rocrand rocsolver

### In your main terminal (not inside the venv), run:
sudo pacman -S hipblaslt
sudo pacman -S hipcub rocprim

### Replace gfxXXXX with your output (e.g., gfx1100)
cmake -DCOMPUTE_BACKEND=hip \
      -DBNB_ROCM_ARCH="gfx1151" \
      -DCMAKE_PREFIX_PATH="/opt/rocm;/opt/rocm/lib/cmake" \
      -S .
	  
### This creates the library file. Now install the package:
### Build using all CPU cores
make -j$(nproc)

### Install into your test_env
pip install ..

python -m bitsandbytes
