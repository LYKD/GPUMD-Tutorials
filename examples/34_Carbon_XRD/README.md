# Powder X-ray diffraction of carbon structures

Calculate powder X-ray diffraction (XRD) with GPUMD for a 128-atom diamond cell, then use the same workflow for AB graphite and three amorphous carbon snapshots. GPUMD calculates intensities at individual reciprocal-space points. The tutorial groups them by scattering angle to produce a powder pattern. You can open the [reference plots](results/xrd-reference.pdf) without installing GPUMD.

## Run diamond first

Use an NVIDIA GPU and a GPUMD version that supports `compute_xrd`. Follow the [standard GPUMD installation instructions](https://gpumd.org/dev/installation.html). The driver uses Python 3. Install Matplotlib in the same Python environment for plotting.

Open a terminal in this example directory and set `GPUMD` to the absolute path of your executable.

```bash
export GPUMD=/path/to/gpumd
python3 run_xrd.py --gpumd "$GPUMD" --case diamond --output diamond.txt
python3 plot_xrd.py diamond.txt diamond.pdf
```

The driver writes 150 bins to `diamond.txt` and reports the strongest bin near 43.9° in 2θ (2.06 Å). Open `diamond.pdf` to see the pattern against d-spacing and 2θ. Use a fresh output name for each run. If GPUMD reports an unknown `compute_xrd` keyword, check which executable `GPUMD` points to. Use `--case graphite --output graphite.txt` for graphite. A strong (002) reflection appears near 26.5° (3.36 Å).

The commands above use GPUMD's built-in neutral-carbon form factor. It is the five-Gaussian fit of Waasmaier and Kirfel for `s = sin(theta)/wavelength` from 0 to 6 Å⁻¹. [Original paper](https://doi.org/10.1107/S0108767394013292)

## Run all five structures

| Case | Atoms | Input |
| --- | ---: | --- |
| Diamond | 128 | [diamond.xyz](structures/diamond.xyz) |
| AB graphite | 6,144 | [graphite.xyz](structures/graphite.xyz) |
| Amorphous carbon, 1.11 g/cm³ | 55,296 | [rho-1.11.xyz](structures/rho-1.11.xyz) |
| Amorphous carbon, 2.83 g/cm³ | 55,296 | [rho-2.83.xyz](structures/rho-2.83.xyz) |
| Amorphous carbon, 3.35 g/cm³ | 55,296 | [rho-3.35.xyz](structures/rho-3.35.xyz) |

```bash
python3 run_xrd.py --gpumd "$GPUMD" --output all-five.txt
python3 plot_xrd.py all-five.txt all-five.pdf
```

The shared [GPUMD input](run.in) uses a zero time step to sample the supplied structure. It sets the wavelength to 1.541838 Å, enables Lorentz–polarization correction, and samples 10–150°. The crystal mesh uses the inverse box lengths. The amorphous mesh uses 0.01 Å⁻¹ along each axis. Allow about 400 MB of temporary disk space per amorphous case.

GPUMD writes raw reciprocal points to `xrd.out` in a temporary directory. The columns are three mesh indices, 2θ in degrees, and intensity per sampled atom. See the GPUMD manual entries `compute_xrd` and `xrd.out` for the equations and sampling conventions.

The driver sums intensities over 150 bins from 10–100° and divides by the total intensity within that range. Its tab-separated table gives the bin center in `two_theta_deg`, its corresponding spacing in `d_angstrom`, and the normalized result in `gpumd_fraction`. It leaves the paper, independent calculation, and LAMMPS columns blank in fresh runs. The full output has 750 data rows. On failure, the driver keeps the failing case directory and saves the completed cases separately.

## Compare with the published spectra

The three amorphous structures are the final frames after 100 ps at 300 K and fixed volume. The `paper_fraction` column of the [archived reference table](results/xrd-reference.txt) contains the values underlying the three amorphous curves in Fig. S6. The `reference_fraction` column was calculated independently from the supplied coordinates using the study's scattering parameters. The two columns agree to six significant figures across the 450 amorphous bins. Both columns are included in the example.

The archived comparison uses the neutral-carbon C RHF fit from
[International Tables for Crystallography, Vol. C, Table 6.1.1.4](https://onlinelibrary.wiley.com/iucr/itc/Cb/ch6o1v0001/table6o1o1o4/).
Fresh GPUMD runs use the Waasmaier–Kirfel fit, so the normalized intensities
can differ slightly. Plot the archived curves with:

```bash
python3 plot_xrd.py results/xrd-reference.txt reference.pdf
```

[Li et al., ACS Nano 20, 11284–11293 (2026)](https://doi.org/10.1021/acsnano.6c00656)
