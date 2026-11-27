# Prismatic Compass Survey Analyzer

A small Flask web app for analysing **closed compass traverses** from a prismatic compass survey. Enter the length and whole-circle bearing of each line. The app works out the closing error, adjusts the traverse with **Bowditch's (compass) rule**, and draws interactive plots of the result.

## Features

- **Dynamic input table**: choose from 2 to 20 traverse lines (A-B, B-C, …). The table comes pre-filled with sample data for a 4-sided traverse.
- **Closing error**: shows the size (m) and the whole-circle bearing (°) of the misclosure.
- **Adjusted data table**: lists, for each line, the original length and bearing, the latitude and departure corrections, and the adjusted length and bearing.
- **Traverse plot**: an interactive Plotly chart that shows
  - the unadjusted traverse (dashed white) and the adjusted traverse (green),
  - a length @ bearing label on every line,
  - dotted lines from each station's unadjusted position to its adjusted position,
  - a red arrow showing the closing error (A' → A).
- **Bowditch plot**: a chart of the correction at each station against cumulative length along the traverse.
- Three tabs (*Input & Results*, *Traverse Plot*, *Bowditch Method*), with pan and scroll-zoom on the plots.

## How it works

For each line with length `L` and bearing `θ` (WCB):

```
Latitude   = L · cos θ
Departure  = L · sin θ
```

For a closed traverse, ΣLat and ΣDep should both be zero. Whatever is left over is the closing error:

```
e        = √(ΣLat² + ΣDep²)
Bearing  = atan2(ΣDep, ΣLat)   (normalised to 0–360°)
```

**Bowditch's rule** shares the error out in proportion to line length:

```
Correction to Lat of a line = −ΣLat · (L / P)
Correction to Dep of a line = −ΣDep · (L / P)
```

Here `P` is the perimeter (the total traverse length). Each station is moved by the cumulative fraction `ΣL / P` of the total correction, so the adjusted traverse closes back on station A.

## Tech stack

| Layer    | Tools                              |
|----------|------------------------------------|
| Backend  | Python, Flask, NumPy               |
| Frontend | HTML, CSS, vanilla JavaScript      |
| Charts   | [Plotly.js](https://plotly.com/javascript/) 2.27 (loaded from CDN) |
| Server   | Gunicorn (for deployment)          |

## Project structure

```
Prismatic_py/
├── app.py              # Flask app: routes and traverse calculations
├── requirements.txt    # Python dependencies
├── templates/
│   └── index.html      # Main page (tabs, input and results layout)
└── static/
    ├── script.js       # Input table, API call, Plotly charts
    └── style.css       # Styling
```

## Getting started

### Prerequisites

- Python 3.8+
- pip

### Installation

```bash
git clone https://github.com/sachi5908/Prismatic_py.git
cd Prismatic_py

# (optional) create a virtual environment
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
```

### Run locally

```bash
python app.py
```

Then open <http://127.0.0.1:5000> in your browser.

### Run in production (Gunicorn)

```bash
gunicorn app:app
```

This also works as the start command on platforms such as Render or Railway.

## Usage

1. Set the **Number of Segments** and click **Generate Table**.
2. Type in the **Length (m)** and **Bearing (WCB °)** of each line.
3. Click **Calculate & Draw Traverse**.
4. Read the closing error and the adjusted table on the *Input & Results* tab. Then open the *Traverse Plot* and *Bowditch Method* tabs to see the plots.

## API

### `POST /calculate`

**Request body:** a JSON array of lines in order (A-B, B-C, …):

```json
[
  { "length": 100.2, "bearing": 45.5 },
  { "length": 120.5, "bearing": 150.0 },
  { "length": 89.8,  "bearing": 220.25 },
  { "length": 110.0, "bearing": 310.75 }
]
```

**Response:**

```json
{
  "error_info":    { "magnitude": "…", "bearing": "…" },
  "plot_data":     { "unadjusted_x": [], "unadjusted_y": [], "adjusted_x": [], "adjusted_y": [], "lengths": [] },
  "bowditch_data": { "perimeter": 0, "cumulative_lengths": [], "error_magnitude": 0 },
  "table_data":    [ { "line": "A-B", "orig_len": "…", "orig_brg": "…", "lat_corr": "…", "dep_corr": "…", "adj_len": "…", "adj_brg": "…" } ]
}
```

If something goes wrong, the endpoint returns `{ "error": "<message>" }` with status `400` (perimeter is zero) or `500`.

## Contributing

Issues and pull requests are welcome. Fork the repo, create a branch, make your changes and open a PR.
