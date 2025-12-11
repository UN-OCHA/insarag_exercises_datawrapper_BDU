# INSARAG USAR exercise sites dataset

This repository is maintained by OCHA’s Brand and Design Unit (BDU) to support the INSARAG team with an up-to-date dataset of global USAR exercise sites.

A GitHub Actions workflow automatically syncs the master Google Sheet into the `usar_sites.csv` file. This CSV is served directly to Datawrapper, ensuring the INSARAG map always reflects the latest updates entered by the team.

## Purpose
Provide a clean and continuously updated data source for the INSARAG USAR exercise sites map and other related visualization products.

## Data visualization
The Datawrapper map using this dataset is available here:  
https://datawrapper.dwcdn.net/dB45i/5/

## Data source
The dataset is synced from a Google Sheet (updated by INSARAG colleagues) using a scheduled GitHub Action.
https://docs.google.com/spreadsheets/d/1MGRNFfmDHl9ZRDEMG_-GVYWvX4KuAQQcjnEnLD16_RQ/edit?usp=sharing

## Owning unit
OCHA Communications Branch (CB)  
Brand and Design Unit (BDU)

## Contact
ochavisual@un.org


---
Geocoding in Google Sheets

**Purpose:** Convert addresses in **Column D** into **latitude (G)** and **longitude (H)** using a custom Apps Script function.

---

## ✅ Script Used

```javascript
/**
 * GEOCODE: Converts an address into [latitude, longitude].
 * Usage:
 *   =INDEX(GEOCODE(D2),1)   // latitude
 *   =INDEX(GEOCODE(D2),2)   // longitude
 */
function GEOCODE(address) {
  if (!address) return ["",""];
  try {
    var res = Maps.newGeocoder().setLanguage('en').geocode(address);
    if (res && res.status === 'OK' && res.results && res.results.length > 0) {
      var loc = res.results[0].geometry.location;
      return [loc.lat, loc.lng]; // vertical array
    }
    return ["",""];
  } catch (e) {
    return ["",""];
  }
}
```

---

## ✅ How to Use

- **Address column:** `D`
- **Latitude column:** `G`
- **Longitude column:** `H`

Formulas:
```gs
=INDEX(GEOCODE(D2),1)   // in G2
=INDEX(GEOCODE(D2),2)   // in H2
```
Drag down for all rows.

---

## ✅ Address Format

Include:
- Street + number
- Postal code
- City
- Country

**Example:**  
`Av. de la Paix 8-14, 1211 Genève, Switzerland`

Add a **comment on D1**:
```
Please enter full address: street + number, postal code, city, country.
Example: Av. de la Paix 8-14, 1211 Genève, Switzerland
```

---

## ✅ Notes

- First-time users must **authorize** the script.
- Quotas apply (Apps Script Maps service).
- After filling lat/lon, use **Paste special → Values** to freeze results.
