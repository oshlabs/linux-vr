Scratchpad:

Some github ticket thread that is informative:
https://github.com/ValveSoftware/SteamVR-for-Linux/issues/610

--

Discord server Linux VR Adventures
wiki: https://lvra.gitlab.io/docs/hardware/

Monado:
* https://monado.freedesktop.org/
* https://monado.freedesktop.org/valve-index-setup.html

* https://gitlab.freedesktop.org/monado/monado

Libsurvive:
* https://gitlab.freedesktop.org/monado/monado

OpenComposite:
* https://gitlab.com/znixian/OpenOVR

--

KERNEL PATCH

patches for vivepro / bigscreen:
* https://github.com/CertainLach/VivePro2-Linux-Driver/blob/master/kernel-patches/0002-drm-edid-parse-DRM-VESA-dsc-bpp-target.patch

Seeing that above patch will apply onto kernel v6.8:
https://github.com/torvalds/linux/blob/v6.8/drivers/gpu/drm/drm_edid.c#L213-L215

* https://github.com/CertainLach/VivePro2-Linux-Driver/blob/master/kernel-patches/0003-drm-amd-use-fixed-dsc-bits-per-pixel-from-edid.patch

This has multiple parts:

First part has some apply_edid_quirks there, but rest seems similar:
https://github.com/torvalds/linux/blob/v6.8/drivers/gpu/drm/amd/display/amdgpu_dm/amdgpu_dm_helpers.c#L123-L127
THIS ONE DOES NOT APPLY WITH ERROR:
drivers/gpu/drm/amd/amdgpu/../display/amdgpu_dm/amdgpu_dm_helpers.c: In function ‘dm_helpers_parse_edid_caps’:
drivers/gpu/drm/amd/amdgpu/../display/amdgpu_dm/amdgpu_dm_helpers.c:125:75: error: ‘struct drm_display_info’ has no member named ‘dp_dsc_bpp’; did you mean ‘max_dsc_bpp’?
  125 |         edid_caps->dsc_fixed_bits_per_pixel_x16 = connector->display_info.dp_dsc_bpp;
      |                                                                           ^~~~~~~~~~
      |                                                                           max_dsc_bpp

Second part should apply cleanly:
https://github.com/torvalds/linux/blob/v6.8/drivers/gpu/drm/amd/display/dc/core/dc_stream.c#L105-L106

Third part adding field to the dc_edid_caps struct should work fine too:
https://github.com/torvalds/linux/blob/v6.8/drivers/gpu/drm/amd/display/dc/dc_types.h#L208-L210

Ok, so clearly, not only the two mentioned patches are necessary as is stated here:

But also the 0002-drm-edid-parse-DRM-VESA-dsc-bpp-target.patch is necessary:
https://github.com/CertainLach/VivePro2-Linux-Driver/blob/master/kernel-patches/0002-drm-edid-parse-DRM-VESA-dsc-bpp-target.patch

--

HOWTO for patching packages:
https://wiki.archlinux.org/title/Patching_packages

Get the package for linux - in this case 6.8.1-arch1
```
pkgctl repo clone --protocol=https linux
cd linux
```

I had to import two keys because otherwise the next step would not proceed:
```
gpg --receive-keys 38DBBDC86092693E
gpg --receive-keys B8AC08600F108CDF
```

Download and extract all source files:
```
makepkg --nobuild
```

Copy the downloaded src/linux-6.8.1 directory into src/linux-6.1.8-patched:
```
cd src
rsync -aHAX linux-6.8.1/ linux-6.8.1-patched
```

Then manually apply the patches from below links onto the new src/linux-6.8.1-patched tree:
* https://github.com/CertainLach/VivePro2-Linux-Driver/blob/master/kernel-patches/0001-drm-edid-Add-Vive-Cosmos-Vive-Pro-2-to-non-desktop-l.patch
* https://github.com/CertainLach/VivePro2-Linux-Driver/blob/master/kernel-patches/0003-drm-amd-use-fixed-dsc-bits-per-pixel-from-edid.patch

Create a unified diff patch:
```
diff --unified --recursive --text linux-6.8.1 linux-6.8.1-patched/ > 0001-bigscreen-beyond.patch
```

Patch the pristine tree with the change to see that it works:
```
cd ../linux-6.8.1
patch -p1 < ../0001-bigscreen-beyond.patch
```

Build the package:
```
cd ../..
makepkg --noextract
```

Install the packages:
```
pacman -U linux-6.8.1.arch1-1-x86_64.pkg.tar.zst linux-headers-6.8.1.arch1-1-x86_64.pkg.tar.zst
```







