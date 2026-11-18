# FCTT-Project-3

Equivalent-circuit modelling of a lithium-ion cell in Jupyter notebooks. The project:

1. builds a Thevenin model (OCV + R0) and checks it against measured discharge data (Part 1),
2. adds an R1–C1 branch with a current-dependent R1 (Part 2),
3. couples the electrical model to a lumped thermal model with Arrhenius temperature-dependent resistances, and fits the heat-transfer coefficient `h` (Part 3).

It also includes a pulse-test parameterisation notebook (R0, R1, C1 against SOC and current) and a demo of physics-based modelling with [PyBaMM](https://pybamm.org/).

## Repository layout

```
.
├── part1.ipynb                          # Part 1: OCV-R0 Thevenin model
├── part2.ipynb                          # Part 2: OCV-R0-(R1||C1) model
├── part3.ipynb                          # Part 3: electro-thermal model and fit of h
├── plots.ipynb                          # Voltage/current plots of the model-training data
├── FCTT_Project3_Parameterisation.ipynb # Extract R0, R1, C1 from pulse data
├── FCTT_Project_3_PyBaMM.ipynb          # PyBaMM DFN pulse simulation demo
├── part1 old.ipynb                      # Older copy of Part 1
├── part2me.ipynb                        # Working copy: splitting pulse data into discharge windows
├── part3 240316-HHMM.ipynb              # Dated snapshots of Part 3 (4 files)
├── data/                                # Input data (see below)
└── LICENSE                              # MIT
```

## Notebooks

### Main notebooks

- **`part1.ipynb`**: Loads the SOC–OCV table and defines `get_OCV` / `get_SOC` interpolation helpers. Estimates R0 from terminal voltage against discharge current at 50% SOC. Takes the discharge segment of `Battery_Testing_Data.csv` (t ≥ 9237 s), tracks SOC by Coulomb counting (2500 mAh capacity), predicts terminal voltage as `v = OCV(SOC) − i·R0`, and reports the RMSE against the measured voltage.
- **`part2.ipynb`**: Same pipeline, with an RC branch added. R0 and C1 are constants, and R1 is a Gaussian-shaped function of the branch current `i_R1`, which is integrated in discrete time (1 s step). Reports the voltage RMSE.
- **`part3.ipynb`**: Extends Part 2 with a lumped thermal model of an 18650 cell (mass, heat capacity, surface area, ambient temperature taken from the rest period of the data). R0 and R1 scale with temperature through Arrhenius terms. Simulates voltage and cell temperature and reports the RMSE of each. A later section fits `h` with `scipy.optimize.minimize` (Nelder–Mead) on the combined temperature and voltage MSE. Note: a `sys.exit(0)` cell stops "Run All" before the optimisation section.
- **`FCTT_Project3_Parameterisation.ipynb`**: Uses `Model_Training_Data_20.csv` to find each current pulse. From the instantaneous voltage jump it computes R0, from the steady-state voltage R1, and from the relaxation time (taken as 4τ) C1. The results are arranged on a SOC (0.9–0.2) × current grid and plotted as 3D surfaces and contour plots.
- **`FCTT_Project_3_PyBaMM.ipynb`**: Simulates a rest / 2C-discharge / rest pulse at 25 °C using the PyBaMM Doyle–Fuller–Newman (DFN) model with the `OKane2022` parameter set and temperature as an input parameter. Plots the voltage components and the lithium concentration profiles.
- **`plots.ipynb`**: Plots voltage and current against time for `Model_Training_Data_{0,20,40}.csv`.

### Snapshots and working copies

These files are kept for reference and are not needed to reproduce the main results:

- `part3 240316-0320.ipynb`, `-0944`, `-1014`, `-1348` are timestamped intermediate versions of Part 3. They show it changing from the Part 1 Thevenin structure to the version with the `h` optimisation.
- `part1 old.ipynb` is an earlier version of Part 1.
- `part2me.ipynb` is an exploratory notebook that splits `Model_Training_Data_20.csv` into its discharge windows.

## Data (`data/`)

| File | Contents | Used by |
|---|---|---|
| `SOC_OCV_MFCTT_2019.csv` | SOC (%) against OCV (`Ecell/V`), tab-separated | Parts 1–3 |
| `SOC_OCV_MFCTT_2019_delim.csv` | Comma-separated copy of the file above. The Part notebooks regenerate it | Parts 1–3 |
| `Battery_Testing_Data.csv` | Time (s), Current (mA), Voltage (V), Temperature | Parts 1–3 |
| `Model_Training_Data_{0,20,40}.csv` | Pulse tests: Time (s), Current (A), Voltage (V). The suffix appears to be the test temperature (°C) | `plots`, Parameterisation, `part2me` |
| `RE__Data_for_Degradation/` | Calendar-ageing (60 °C, 100% SOC) and cycle-ageing (45 °C, 1C) capacity and resistance data (`.xlsx`, `.csv`) | Not used by any notebook |

## Installation

Python 3.9 was used for most notebooks (the PyBaMM notebook was saved with 3.11).

```bash
pip install numpy pandas matplotlib scipy jupyter
# for FCTT_Project3_Parameterisation.ipynb
pip install seaborn tabulate
# for FCTT_Project_3_PyBaMM.ipynb (the notebook also runs `%pip install pybamm`)
pip install pybamm
```

Some cells use `%matplotlib Qt` for interactive windows. These need a Qt backend (e.g. `pip install PyQt5`), or you can skip them and keep `%matplotlib inline`.

## Running

Start Jupyter from the repository root, because the notebooks load data with relative paths like `./data/...`:

```bash
jupyter notebook
```

Then open the notebooks in order: `part1` → `part2` → `part3`. The Parameterisation, PyBaMM and `plots` notebooks are independent of the Part notebooks.

## License

MIT. See [LICENSE](LICENSE).
