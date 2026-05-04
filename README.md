# Bus Passenger Counter

A lightweight, browser-based passenger counting tool for transit drivers. Designed to run on a tablet or phone mounted in the cab, it lets a driver tally boardings and exits during a shift, optionally tag each entry with GPS coordinates and the current stop, and export the full activity log as a CSV at the end of the shift.

## How It Works

This tool is a **single HTML file** that runs entirely in the device's browser. There is no server, no backend, and no account to sign in to. Open the page once on the device, and the driver works with it directly throughout the shift.

All shift activity (passenger counts, log entries, GPS readings) is held in the browser's memory while the app is open. **At the end of the shift, the driver taps "End Shift" to download a CSV file** containing the complete record of the day's activity. That CSV is the deliverable. Until that export happens, the data lives only on the device.

A few preferences (bus number, driver name, route) are saved to the browser's local storage so the driver doesn't have to retype them every shift, but the activity log itself is intentionally not persisted across page reloads. This keeps each shift self-contained and avoids accidentally mixing entries from different days.

## Features

### Trip Information

Three required fields at the top of the screen identify the shift:

- **Bus #** ‑ a dropdown of available buses
- **Driver Name** ‑ free text field
- **Route** ‑ a dropdown of routes serviced

All three must be filled in before any boarding action can be logged. If a driver taps a button without completing these fields, the missing fields are highlighted in red and an alert lists what's needed. Once a value is entered, it persists across page reloads on that device so the driver doesn't have to retype it for every shift.

### Passenger Boarding and Exit

Two large, color-coded buttons are the primary controls:

- **+ Board** (green) ‑ increments the on-bus count and logs a boarding entry
- **‑ Exit** (red) ‑ decrements the on-bus count and logs an exit entry

The current passenger count is displayed prominently above the buttons. The Exit button automatically disables when the count reaches zero so the driver can't accidentally log a negative count.

### Wheelchair and Bicycle Counters

Side counter cards on the left track accessibility-related boardings independently:

- **Wheelchairs** ‑ tap to log a wheelchair boarding
- **Bicycles** ‑ tap to log a bicycle being loaded

Each tap increments the running total displayed on the card and adds an entry to the activity log. These counts are tracked separately from the regular passenger count.

### Stop Navigation

When a route is selected, the left sidebar shows the current stop. Drivers can navigate forward and backward through the route's stop list using the arrow buttons. The displayed stop is automatically attached to any boarding or exit entries the driver logs while at that stop.

If GPS is enabled (see below), the stop sidebar is hidden because GPS coordinates are assumed to be the location source for entries.

### GPS Tracking (Optional)

A driver can tap **Enable GPS** to begin continuous location tracking. When active:

- Each log entry is automatically tagged with latitude, longitude, and accuracy (in meters)
- A status indicator shows the current GPS state with a colored dot (gray = off, green = active, red = error)
- Coordinates appear in the activity log as clickable Google Maps links

GPS is fully optional. When it's off, entries are tagged with the manually selected stop instead. The driver can toggle GPS on or off at any time during the shift.

### Activity Log

Every boarding, exit, wheelchair, and bicycle entry is recorded in a chronological log displayed at the bottom of the screen (newest first). Each entry shows:

- The action type (Boarded, Exited, Wheelchair, Bicycle)
- The timestamp
- Either GPS coordinates (clickable to open in Google Maps) or the current stop name, depending on what was active when logged

### Undo

The **Undo** button removes the most recent log entry and reverses its effect on whichever counter it incremented. This provides a quick correction path for miscounts. Undo applies to the most recent action only; it is not a multi-level history.

### Light and Dark Themes

A theme toggle switches between light and dark color schemes. The selection is remembered across reloads, and the app respects the device's system preference on first load.

### End of Shift Export

When the shift ends, the driver taps **End Shift**. The app:

1. Verifies the bus is empty (passenger count must be zero)
2. Asks the driver to type "End shift" to confirm (prevents accidental taps)
3. Generates a CSV file with one row per log entry
4. Triggers a browser download of the CSV
5. Resets all counters and clears the log for the next shift

The CSV filename follows the pattern `<Route>_<Bus>_<YYYY-MM-DD>.csv` and includes the following columns:

| Column | Description |
|---|---|
| Bus # | Bus identifier from the trip info |
| Driver | Driver name from the trip info |
| Route | Route from the trip info |
| Stop # | Stop ID, if a stop was selected at the time of the entry |
| Stop | Stop name, if a stop was selected |
| Action | Boarded, Exited, Wheelchair, or Bicycle |
| Time | Local time of the entry |
| ISO Timestamp | Machine-readable UTC timestamp |
| Latitude | GPS latitude, if GPS was active |
| Longitude | GPS longitude, if GPS was active |
| Accuracy (m) | GPS accuracy in meters, if GPS was active |

## Where the Data Lives

This is worth restating because it shapes how the tool should be used operationally.

The app **runs locally in the device's browser**. Nothing is transmitted to a server during the shift. All passenger counts, log entries, and GPS readings exist only in the browser's memory until the driver exports the CSV at end of shift. **The CSV download is the only way data leaves the device.** If the driver closes the browser, refreshes the page, or the device loses power before tapping End Shift, the unsaved log for that shift is lost.

Drivers should be trained to:

- Tap **End Shift** at the end of every shift to download the CSV
- Save or upload that CSV to wherever the agency collects ridership data
- Avoid closing or refreshing the browser tab mid-shift

The trip info fields (bus, driver, route) and the theme preference are the only items saved to the browser between sessions, via the browser's local storage. These are conveniences, not records of activity.

## Customizing for a Specific Agency

The tool ships as a generic template with three placeholder buses, three placeholder routes, and three placeholder stops per route. To deploy it for a specific agency, edit two areas of `index.html`:

1. **The bus and route dropdowns** in the HTML, replacing the `Bus 1 / Bus 2 / Bus 3` and `Route 1 / Route 2 / Route 3` options with the agency's actual fleet and route names.
2. **The `stopsByGroup` and `routeToGroup` objects** in the JavaScript, replacing the placeholder stops with the agency's real stop lists. Each stop has an `id` (used in the CSV export) and a `name` (shown to the driver). If multiple route variants share a stop list, they can map to the same group key.

The app's logic, layout, and styling do not need to change for a new agency, only the data.

## Browser and Device Requirements

- A modern browser (Chrome, Safari, Firefox, or Edge from the last few years)
- Touchscreen recommended for in-cab use
- For GPS tracking: a device with location services and the browser's location permission granted to the page
- For local storage of trip info and theme: cookies and site data must be enabled

The app does not require an internet connection during the shift once the page has been loaded.
