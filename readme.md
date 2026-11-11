# Treck Traffic

**Scheduled traffic-data collection for transportation research.**

Treck Traffic collects repeated travel-time and distance observations for a study area. Researchers supply origin–destination pairs and an observation schedule; the application prepares and runs requests over a selected period, stores responses in PostgreSQL, and makes the accumulated data available for download as Excel or JSON.

The tool was developed during research work at **Kwame Nkrumah University of Science and Technology (KNUST), Ghana**. It was used by faculty in **Geography and Civil Engineering** and by **research students** to investigate traffic conditions over routes and time periods without repeatedly collecting observations by hand.

This repository contains the research software. It is not a published traffic dataset or a claim that any particular number of observations was collected.

## Research workflow

1. **Define the study area.** Supply origin–destination pairs, with identifiers and geographic coordinates, in an Excel workbook.
2. **Choose the observation period.** Specify study dates, eligible weekdays, collection windows, and sampling intervals.
3. **Build the schedule.** The application groups requests by observation time and schedules them for the relevant origin–destination pairs.
4. **Collect observations.** Scheduled jobs call the Google Maps Directions API and extract origin/destination information, distance, normal journey duration, and traffic-adjusted duration.
5. **Store and retrieve results.** Responses are saved through Sequelize to PostgreSQL and can be exported through the application's Excel and JSON download routes.

This supports repeated observations for transportation and spatial research. The stored fields and schedules provide the basis for later analysis; this application does not itself establish a statistical traffic model or guarantee complete collection when the API or host is unavailable.

## What is in the code

| Component | Responsibility |
| --- | --- |
| [`index.js`](index.js) | Web interface, study setup, scheduling lifecycle, and download endpoints |
| [`preprocessInput.js`](preprocessInput.js) | Read the study workbook; interpret trip IDs, coordinate pairs, time windows and intervals; prepare dated requests |
| [`cron.js`](cron.js) | Schedule and dispatch time-specific request batches |
| [`request.js`](request.js) | Request traffic information and persist returned observations |
| [`models/Response.js`](models/Response.js) and [`database/dbconn.js`](database/dbconn.js) | PostgreSQL model and connection |
| [`views/index.pug`](views/index.pug) | Simple research-operator interface |

The application exposes a start action, workbook upload, and routes to retrieve JSON or Excel data. The legacy archive-related files and routes are retained, but the automatic archive job in the current `index.js` is commented out. **Scheduled collection and PostgreSQL storage are separate from automatic archival**, which should not be described as a current operational guarantee.

## Data model

Each successfully stored observation includes:

- origin and destination
- request / trip identifier
- scheduled departure time
- distance
- journey duration
- traffic-adjusted journey duration

See [`models/Response.js`](models/Response.js) for the actual schema. The database preserves observations for subsequent export; it does not by itself validate representativeness, accuracy of third-party responses, or completeness of a study period.

## Running locally

This is a historical research prototype, not a hardened public service. Run it in a trusted development environment only.

**Requirements:** Node.js and npm, a PostgreSQL database, a Google Maps API key authorized for the relevant traffic requests, and a study workbook matching the expected input structure.

```bash
git clone https://github.com/teckedd-code2save/automate-data-archiving.git
cd automate-data-archiving
npm install
```

Provide local configuration through a private `.env` file. The code currently reads the following variables:

```dotenv
google_api_key=YOUR_GOOGLE_MAPS_API_KEY
local_db=postgres://USER:PASSWORD@HOST:5432/DATABASE
live_db=postgres://USER:PASSWORD@HOST:5432/DATABASE
```

The current database module selects `live_db` when `PORT` is set and `local_db` otherwise. Check that behavior before running in your own environment.

Prepare `input/input_data.xlsx` with the study's origin–destination pairs, calendar and trip-time settings. The ingestion code expects four worksheets (commonly `optional_inputs`, `od_pairs`, `calendar`, `trip_times`). Review [`preprocessInput.js`](preprocessInput.js) and the supplied historical input workbook for the actual column conventions. Do not reuse historical coordinates or project data without permission.

```bash
npm start
```

The Express app defaults to port `5000`. The interface includes an input-upload action and a start action. **Starting a run schedules real, potentially billable API requests.** Test with a small, authorized study before expanding collection.

### Available routes in the current source

| Route | Purpose |
| --- | --- |
| `/` | Research-operator interface |
| `POST /upload` | Upload the study workbook |
| `GET /start` | Initialize a run and schedule requests |
| `GET /output` | Download stored observations as Excel |
| `GET /backup` | Download stored observations as JSON |
| `GET /archives` | Historical archive download path; availability depends on archived files |

These routes are documented from the source; they have not been independently exercised as part of this README revision.

## Scope and limitations

- **Research prototype.** Upload and run endpoints are not designed as a secure public-facing API. Use a trusted environment and appropriate network restrictions.
- **Collection is dependent on external APIs.** Rate limits, billing, unavailable traffic fields, time-zone handling and failed requests can affect the resulting dataset.
- **No claimed dataset scale.** The code is designed for repeated scheduled observations; this README does not assert an unverified row count, uptime target or automated recovery guarantee.
- **Exports are implemented.** Automatic end-of-day archive creation is not enabled in the current entry point.
- **Reproduction requires configuration and review.** The repository includes historical input/output artifacts and a Dockerfile whose entry-point path should be checked against the current root-level `index.js`. The package's `test` script is still a placeholder.
- **Research provenance.** A transportation-data collection tool and the separate peer-reviewed polling-station geocoding study are distinct pieces of the author's research experience. Do not conflate their methods or results.

## Research context

Developed by **Edward Kwabena Twumasi** during his Teaching and Research Assistant work at KNUST (2021–2022). The software addressed an operational research need: obtaining repeated, time-specific traffic observations across many origin–destination pairs so that faculty and student researchers could analyse transportation conditions over time.

Related research publication (separate from this tool):

Dumedah, G., Binche, N. N.-Y., Bob-Milliar, G. M., Iddrisu, S., Twumasi, E. K., & Boateng, J. A. (2023). *The case of electoral polling station data for geocoding in facilitating accessibility to social, economic and cultural opportunities in Ghana*. **African Geographical Review**, 42(5), 560–573. [DOI: 10.1080/19376812.2022.2076134](https://doi.org/10.1080/19376812.2022.2076134). First published online 19 May 2022.
