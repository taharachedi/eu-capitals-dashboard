# Reflections — TD1

## 1. Personal weighting (weather vs CO2)
I would not ignore either factor, but I would give slightly more weight to CO2
(about 60% CO2 / 40% weather): a sunny week is nice, but a long flight for a
short vacation has a real, lasting climate cost. A destination close to Paris
with decent weather would be my ideal choice.

## 2. Why one batched API call is more reliable
Each separate request opens its own connection (TLS handshake), so 26 calls mean
26 independent chances for a timeout, a hang or rate-limiting, especially from a
shared GitHub Actions runner IP. The probability that *all* 26 succeed is much
lower than the probability that one single request succeeds. One batched call
means one connection: either everything arrives, or the retry logic handles a
single failure cleanly, with no half-filled ranking.

## 3. What to do about the CO2 emissions factor
Before publishing for real users, I would check the latest official DEFRA (UK
government) conversion factors for short-haul flights, cite the exact year and
table in the README, and decide whether to include radiative forcing (non-CO2
effects), which can roughly double the impact. I would also store the factor
with its source and date in the JSON, so every result is traceable, and review
it every year when new factors are published.

## 4. Normalization: KPI vs raw measurement
Adding a very cold, rainy capital would not change any other city's real
temperature, but it would change their scores, because normalize() rescales
values relative to the min and max of the current list. The raw measurement
(°C, mm, kg CO2) is an absolute fact; the score is a relative construct that
depends on which cities are compared. A KPI is therefore a design choice, not a
neutral fact, and it can change even when reality does not.

## 5. Predicted winning capital
I predict **Madrid**. In late October, southern capitals have clearly better
weather (warmer, sunnier, drier), and Madrid is only about 1,050 km from Paris,
much closer than Athens, Valletta or Nicosia. With a 50/50 formula, it should
keep a good CO2 score while having one of the best weather scores.

## 6. Was my prediction right?
Yes: the dashboard recommended **Madrid, Spain** (weather score 0.81, CO2 score
0.71, 315.9 kg CO2e round trip, 1,052.9 km each way), which matches my
prediction. This shows the 50/50 formula rewards a "good compromise" city rather
than the best city on one criterion: on the scatter plot, the destination with
the best weather (score close to 1.0) is far from Paris, so its poor CO2 score
pulls its final score down, while Madrid is good on both axes at once.

## 7. Signal that the pipeline silently broke
The only visible signal is the "Data last updated …" timestamp at the bottom of
the page: if it is more than a week old, the Monday job has stopped or is
failing. On its own, this is weak, because it is small, at the bottom of the
page, and visitors must compute the age themselves. I would add a visible
warning banner when generated_at is older than 8 days, and enable email
notifications for failed workflow runs.

## 8. Origin-city extension
I changed two files. In scripts/fetch_and_score.py, I replaced the hardcoded
Paris coordinates with ORIGIN_NAME / ORIGIN_LAT / ORIGIN_LON (London), used
ORIGIN_NAME in the exclusion filter, and wrote "origin" into rankings.json. In
docs/index.html, I read data.origin to display the city name instead of the
hardcoded "Paris", and made the chart title use dest.length (27 destinations
now, since London is not in the list, so Paris becomes a destination). These
were the right places because the script owns the data and the page should
only display what the data says. I also added a cache-busting parameter
(?t=Date.now()) to the fetch, because the page first kept showing the old
Paris data from the browser/CDN cache.

The "best" destination is not absolute. From London, Madrid still wins
(0.741, 1,263 km instead of 1,053 km), but the rest of the ranking changed:
Paris enters at 3rd place and Dublin climbs to 4th because they are very close
to London, while Rome and Valletta drop. The weather is identical; only the
distances, and therefore the normalized CO2 scores, depend on the origin.
