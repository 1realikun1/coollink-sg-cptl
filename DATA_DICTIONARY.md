# Data dictionary and scope

All CSV files are UTF-8 with a header row. Times marked SGT carry UTC+08:00 (Asia/Singapore). Spatial layers use EPSG:3414 unless the GeoPackage layer metadata explicitly identifies another CRS; geographic station latitude/longitude are degrees. Do not infer CRS from coordinate magnitudes.

## Main analysis

`data/processed/routing/behavior_informed_od_sample.csv`: 152 synthetic origin–destination pairs. `region_id` identifies the district, `od_id` the synthetic pair; `origin_node` and `destination_node` reference the archived OSM network. `od_probability` in departure-specific route tables is a synthetic weight, not an observed frequency.

`behavior_informed_probability_candidates.csv`: 12,122 candidate route–case records.
`behavior_informed_probability_recommendations.csv`: 760 selected route–case records (Clementi 200, Jurong East 160, Toa Payoh 200, Raffles Place 200).
Case key: (`region_id`, `od_id`, `departure_time_sgt`). Candidate key adds `route_id`.

| Field | Meaning / unit |
|---|---|
| `route_length_m`, `travel_time_min` | Route distance in m, walking time in min (1.2 m/s) |
| `route_mean_utci_deg_c`, `route_maximum_utci_deg_c` | Modeled equivalent temperature, °C |
| `cptl_above_26_deg_c_min` (also 32, 38) | Traversal-time integral of max(UTCI − threshold, 0), °C·min |
| `departure_snapshot_sun_distance_m`, `departure_snapshot_shade_distance_m` | Departure-time sun/shade composition used by the behavior model; m |
| `mean_acceptance_probability`, `acceptance_probability_low/high` | External model mean and bootstrap interval; fraction 0–1 |
| `physical_detour_percent` | Distance increase relative to shortest route, % |
| `cptl_reduction_deg_c_min`, `cptl_reduction_percent` | Saving relative to shortest route, °C·min or % |
| `probability_weighted_cptl_reduction_*` | Predicted acceptance multiplied by CPTL saving; not observed adoption |
| `is_shortest_distance` | True when shortest-distance reference is retained |
| `selected_behavioral_offer` | Selected row marker (including retained reference); consult `is_shortest_distance` to count alternative offers |

An alternative must have positive modeled CPTL saving and mean acceptance ≥0.05. Selection maximizes probability × absolute saving within the archived candidate pool. Weighted medians use the first sorted value whose cumulative positive weight reaches 50%. Median acceptance is conditional on alternatives being offered; other summary medians include retained shortest routes.

`behavior_informed_probability_segments.csv` stores route-segment exposure and timing; join through route/case keys present in its header and the stable `segment_id`. The route GeoPackage stores candidate and recommended geometry. `outputs/tables/behavior_informed_probability_summary.csv` contains the four district summaries.

## Controls and supplementary materials

`four_region_behavior_probability_*`: separate multistation network-coverage probability panel (240 cases).
`four_region_multistation_dynamic_cptl*`: corresponding fixed-distance-cap routing controls.
`behavior_informed_od_routes.*`: fixed-cap evaluation of the primary synthetic OD sample; not the probability offer estimand.
`outputs/manuscript/revision_v2/supplementary/table_S1_case_level_primary_5pct.csv`: 240 fixed 5% legacy comparator cases; S2 contains departure summaries, S3 OD summaries, and S4 540 weather cases. These legacy controls use the historical single-station background documented in the manuscript.
`outputs/manuscript/revision_v2/tables/`: archived table-generation inputs (historical filenames are retained; current article numbering is defined by the manuscript).
`outputs/tables/temporal_thinning_*.csv`: 30/60/120 min retention of a 37-timestamp, 15 min multistation field; fixed candidate pool and 240 control cases. Timestamp reductions are count savings, not measured computational runtimes.
`figure_07_route_examples/`: final route-selection provenance and an additional Jurong East illustration. Historical filenames still say Fig6; the formatted manuscript calls it Figure 7. The additional OD is excluded from all aggregate panels.

## Environmental and calibration inputs

`data/processed/network/`: four directed pedestrian networks, with `nodes` and `edges` layers and OSM attribution.
`data/processed/heat/four_region_multistation_idw/`: four segment-by-time thermal tables, 37 quarter-hour timestamps from 09:00 to 18:00 SGT on 17 August 2026. Total 2,481,072 rows. `road_mean_utci_deg_c` and other thermal columns are model outputs; `road_mean_sun_fraction` is a fraction in [0,1]. Raw forcing, radiation and urban geometry assumptions remain in the source configurations.
`data/processed/weather/`: NEA station observations, four-nearest-station IDW backgrounds and leave-one-station-out diagnostics. Air temperature is °C, relative humidity %, processed wind speed m/s; original station wind observations may retain knots in explicitly named source fields.
`data/processed/behavior/`: fitted external choice model (β=1.222970067642403, τ=23.49358385960436 m) and 500 participant-cluster bootstrap parameter replicates. The 46-participant/408-choice records remain available at their original source.

The archive verifies numerical summaries and preserves processed inputs; it does not establish local behavior calibration, in-field UTCI accuracy, or health benefit.
