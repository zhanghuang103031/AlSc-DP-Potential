# Deep Potential Model for the Al–Sc Binary System

This repository provides a Deep Potential (DP) interatomic potential for the binary Al–Sc system. The model was trained using density-functional-theory (DFT) reference data and the DP-GEN active-learning workflow, and is intended for atomistic simulations of Al–Sc alloys, including fcc Al, hcp Sc, L1₂-Al₃Sc, Al–Sc solid-solution configurations, defects, grain boundaries, surfaces/slab-like low-coordination environments, and Al/Al₃Sc interfacial structures.

## Short description

A DeePMD-kit interatomic potential for the binary Al–Sc alloy system, trained with DFT data and DP-GEN active learning for large-scale molecular dynamics simulations of Al–Sc defects, interfaces, precipitates, and deformation behavior.

## Repository contents

```text
.
├── README.md
├── model/
│   ├── graph.pb                  # frozen DP model
│   └── graph-compress.pb         # compressed DP model, optional
├── input/
│   ├── input.json                # training input file, if released
│   └── type_map.raw              # element type map
├── examples/
│   └── lammps/
│       └── in.al_sc_dp           # example LAMMPS input
└── data/                         # optional representative training/test data
```

The exact released file names may differ. Please check the `model/` and `examples/` directories.

## Model scope

The model is trained and validated for the **binary Al–Sc system**. It is suitable for simulations involving:

- fcc Al and hcp Sc;
- Al–Sc solid-solution configurations;
- ordered L1₂-Al₃Sc;
- selected vacancies, interstitials, and antisite-like local environments;
- Al grain boundaries and Al/Al₃Sc interfacial configurations;
- large-scale DP-MD simulations of deformation and dislocation–precipitate interactions.

## Limitations

This model should not be directly applied to multicomponent Al alloys containing additional elements such as Mg, Cu, Zr, Ti, or other solutes without additional training and validation. Extension to systems such as Al–Sc–Zr would require new DFT reference data describing interactions among the added solute elements, Al, and Sc, as well as their coupling with defects, interfaces, and precipitates.

Some properties may be more sensitive than others to the training database, reference-state choices, and validation protocol. Users should validate the model for their specific target application before production simulations.

## Software compatibility

The model was prepared for use with DeePMD-kit 2.x and LAMMPS compiled with the DeePMD pair style. The frozen TensorFlow model is typically used as:

```lammps
pair_style deepmd graph.pb
pair_coeff * *
```

For the compressed model:

```lammps
pair_style deepmd graph-compress.pb
pair_coeff * *
```

Make sure the element order in LAMMPS matches the type map used during training. For this model, the intended type map is:

```text
Al Sc
```

## Example LAMMPS usage

A minimal LAMMPS input is provided in `examples/lammps/in.al_sc_dp`.

```lammps
units           metal
boundary        p p p
atom_style      atomic

read_data       data.al_sc

pair_style      deepmd ../../model/graph.pb
pair_coeff      * *

mass            1 26.9815385   # Al
mass            2 44.955908    # Sc

neighbor        2.0 bin
neigh_modify    every 1 delay 0 check yes

timestep        0.001
thermo          100
thermo_style    custom step temp pe ke etotal press

velocity        all create 300.0 12345 mom yes rot yes dist gaussian
fix             1 all nvt temp 300.0 300.0 0.1
run             10000
unfix           1
```

## Freezing and compressing the model

If the model is released from a training checkpoint, freeze it first:

```bash
dp freeze -o graph.pb
```

Then compress the frozen model:

```bash
dp compress -i graph.pb -o graph-compress.pb
```

After compression, always test the compressed model against the same validation/test structures used for the frozen model:

```bash
dp test -m graph.pb -s path/to/test_system -n 1000 -d test_frozen
dp test -m graph-compress.pb -s path/to/test_system -n 1000 -d test_compressed
```

The compressed model should be used only if its energy and force errors are essentially unchanged for the target validation systems.

## Citation

If you use this model, please cite the associated manuscript:

```text
[Author list], "Training and validation of the interatomic potential for Al–Sc alloy based on deep learning framework", Journal of Materials Science, [year], [DOI].
```

Please also cite DeePMD-kit and DP-GEN where appropriate.

## Contact

For questions about the model or training data, please contact:

```text
Liang Zhang
Chongqing University
Email: liangz@cqu.edu.cn
```
