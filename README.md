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


--------

# Geocoding in Google Sheets

Purpose: Convert addresses in Column D into latitude (G) and longitude (H) using a custom Apps Script function.

## Script used

```javascript

// --- CONFIG ---
const SHEET_NAME = 'USAR_data'; // <-- your tab name
const DECIMALS = 6;             // round lat/lon to 6 decimals
// --- END CONFIG ---

function bakeLatLon() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(SHEET_NAME);
  if (!sheet) return;

  var lastRow = sheet.getLastRow();
  if (lastRow < 2) return;

  var addrRange = sheet.getRange(2, 4, lastRow - 1, 1); // D2:D (Address)
  var latRange  = sheet.getRange(2, 7, lastRow - 1, 1); // G2:G (Lat)
  var lonRange  = sheet.getRange(2, 8, lastRow - 1, 1); // H2:H (Lon)

  var addresses = addrRange.getValues();
  var lats = latRange.getValues();
  var lons = lonRange.getValues();

  var cache = {};
  for (var i = 0; i < addresses.length; i++) {
    var addr = (addresses[i][0] || '').toString().trim();
    if (!addr) continue;

    // Skip rows where lat & lon are already numeric values
    var hasLat = typeof lats[i][0] === 'number';
    var hasLon = typeof lons[i][0] === 'number';
    if (hasLat && hasLon) continue;

    // Geocode and write numeric values (no formulas)
    var coords = cache[addr] || geocodeAddress_(addr);
    cache[addr] = coords;

    if (coords && coords.lat != null && coords.lng != null) {
      lats[i][0] = Number(coords.lat.toFixed(DECIMALS));
      lons[i][0] = Number(coords.lng.toFixed(DECIMALS));
    }
    Utilities.sleep(120); // gentle pacing
  }

  // Write back values
  latRange.setValues(lats);
  lonRange.setValues(lons);
}

function geocodeAddress_(address) {
  try {
    var res = Maps.newGeocoder().setLanguage('en').geocode(address);
    if (res && res.status === 'OK' && res.results && res.results.length > 0) {
      var loc = res.results[0].geometry.location;
      return { lat: loc.lat, lng: loc.lng };
    }
  } catch (e) {}
  return { lat: null, lng: null };
}
```

I added an hourly trigger in the Google Sheet so the script automatically recalculates the latitude and longitude values. This ensures that any newly added or updated addresses in column D are processed without requiring manual action. The trigger runs the geocoding function in the background once per hour, keeping the dataset up to date for export and use in Datawrapper.

## How to use

Address column: D  
Latitude column: G  
Longitude column: H  

Formulas:

```
=INDEX(GEOCODE(D2),1)   // G2 latitude
=INDEX(GEOCODE(D2),2)   // H2 longitude
```

Drag down for all rows.

## Address format

Include:
- Street and number
- Postal code
- City
- Country

Example:  
Av. de la Paix 8-14, 1211 Genève, Switzerland

Comment to add in D1:

```
Please enter full address: street + number, postal code, city, country.
Example: Av. de la Paix 8-14, 1211 Genève, Switzerland
```

## Notes

- First-time users must authorize the script.
- Quotas apply (Apps Script Maps service).
- After filling latitude and longitude, use Paste special → Values to freeze results.
