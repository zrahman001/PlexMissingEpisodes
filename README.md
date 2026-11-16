# Plex Missing Episodes (TVMaze)

A PowerShell script that compares the episodes recorded in a Plex TV library with the episode listings in [TVMaze](https://www.tvmaze.com/) and reports episodes that appear to be missing from Plex.

The script uses TVMaze's public API, so **no TVMaze account, API key, or PIN is required**. It uses Plex's TVDB and IMDb identifiers where available and falls back to a conservative show-title match when necessary.

## Features

- Scans the TV-show sections in a Plex Media Server library.
- Checks every show, or limits the scan to a show name supplied with `-SingleShowFilter`.
- Excludes selected shows with `-IgnoreShows`.
- Resolves Plex TVDB IDs and IMDb IDs to TVMaze show records. If neither identifier resolves, it attempts an exact, normalised title match, using the premiere year when Plex provides one.
- Compares Plex episodes with TVMaze episode metadata using air dates, season and episode numbers, and episode titles.
- Includes handling for daily shows and date-based episode naming.
- Can include Season 0 specials/extras.
- Supports detailed and compact report formats, console output, and saving the report to a file.
- Caches TVMaze show mappings and episode data locally to reduce repeat API requests.

## Requirements

- Plex Media Server reachable from the computer running the script.
- A Plex account and either a Plex authentication token or Plex username and password.
- Internet access to `api.tvmaze.com`.
- Windows PowerShell 5.1 is the recommended runtime.

The script has been designed for a Plex TV library. It does not scan movie libraries.

## Download and setup

1. Download `PlexMissingEpisodes-TVmaze.ps1` from this repository.
2. Place it in a folder where it can create and update its cache file.
3. Make sure the computer running the script can reach your Plex server and the TVMaze API.
4. Open PowerShell and run the script with your Plex server URL and authentication details.

The Plex server URL should include the protocol, host name or IP address, and port. Do not add a trailing slash.

Example:

```powershell
http://192.168.1.10:32400
```

### Plex authentication

You can authenticate in either of two ways:

- **Plex token:** Supply `-PlexToken`. This bypasses the script's Plex username/password sign-in step.
- **Plex username and password:** Supply both `-PlexUsername` and `-PlexPassword`. The script requests a Plex authentication token from Plex.tv before connecting to your server.

A Plex token grants access to your Plex account and should be treated like a password. Do not commit tokens or passwords to the repository, put them in shared scripts, or publish them in screenshots or logs.

## Usage

Run the script from the folder where it was downloaded. Replace the example server address and token with your own values.

### Scan all TV shows

```powershell
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN"
```

### Scan one show

`-SingleShowFilter` performs a partial, case-insensitive name match. Only matching shows are processed.

```powershell
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -SingleShowFilter "Breaking Bad"
```

### Ignore selected shows

`-IgnoreShows` accepts an array of show names. The names are compared with the Plex library show titles. The ignore list is applied when `-SingleShowFilter` is not specified.

```powershell
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -IgnoreShows "Show Name One", "Show Name Two"
```

### Include specials and extras

By default, Season 0 is excluded. Add `-IncludeExtras` to request TVMaze specials and include Season 0 episodes in the comparison.

```powershell
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -IncludeExtras
```

### Use compact output

Add `-SimpleOutput` to request one line per missing episode instead of the detailed report.

```powershell
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -SimpleOutput
```

### Save the report to a file

Use `-OutputFile` to save the report. The destination folder must already exist.

```powershell
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -OutputFile "C:\Reports\Plex-Missing-Episodes.txt"
```

### Refresh cached episode data

The script keeps episode data in a local cache. Use `-CacheRetentionDays` to set how long episode data may be reused, or `-ForceRefresh` to bypass the episode cache for the current run.

```powershell
# Reuse episode data only while it is less than 3 days old
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -CacheRetentionDays 3
```

```powershell
# Fetch episode data again, ignoring the existing episode cache
.\PlexMissingEpisodes-TVmaze.ps1 `
    -PlexServer "http://192.168.1.10:32400" `
    -PlexToken "YOUR_PLEX_TOKEN" `
    -ForceRefresh
```

`-ForceRefresh` bypasses the cached episode lists. Cached Plex-ID-to-TVMaze-ID mappings are still reused.

## Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `-PlexServer` | String | Yes | — | Plex Media Server base URL, for example `http://192.168.1.10:32400`. |
| `-PlexUsername` | String | No* | Empty | Plex account username, used with `-PlexPassword` when no token is supplied. |
| `-PlexPassword` | String | No* | Empty | Plex account password, used with `-PlexUsername` when no token is supplied. |
| `-PlexToken` | String | No* | Empty | Plex authentication token. When supplied, the username/password login step is skipped. |
| `-SingleShowFilter` | String | No | Empty | Process only show titles containing this text. |
| `-IgnoreShows` | String array | No | Empty | Show titles to skip when scanning the full library. |
| `-OutputFile` | String | No | Console output | File path for the report. The parent directory must exist. |
| `-SimpleOutput` | Switch | No | Off | Use compact, one-line-per-episode output. |
| `-IncludeExtras` | Switch | No | Off | Include TVMaze specials and Season 0 episodes. |
| `-CacheRetentionDays` | Integer | No | `1` | Maximum age, in days, for cached episode data to be reused. |
| `-ForceRefresh` | Switch | No | Off | Bypass the cached episode lists and retrieve them again from TVMaze. |

\* Supply either `-PlexToken` or both `-PlexUsername` and `-PlexPassword`.

## How show matching works

For each Plex show, the script attempts to identify the corresponding TVMaze show in this order:

1. Plex TVDB ID via TVMaze's show lookup endpoint.
2. Plex IMDb ID via TVMaze's show lookup endpoint.
3. A conservative title search. The title must match after normalisation; when Plex provides a year, the TVMaze premiere year must also match. The fallback is accepted only when it identifies one unambiguous result.

Shows that cannot be resolved are skipped and reported in the script's console output. A skipped show is not proof that it has no missing episodes; it means the script could not safely identify its TVMaze record.

## How episode comparison works

The script reads the episodes Plex knows about and compares them with TVMaze's episode list. It uses air dates where available, then season/episode numbers and episode titles as matching signals. It also attempts to derive episode numbers and dates from media filenames when Plex metadata is incomplete, which helps with some daily shows and date-based naming conventions.

The script ignores episodes that:

- Have not aired more than 24 hours before the scan.
- Have no usable season, episode number, or air date in the TVMaze data.
- Belong to Season 0 when `-IncludeExtras` has not been specified.

TVMaze's standard show-episodes endpoint excludes specials by default. The script requests the `specials=1` variant when `-IncludeExtras` is used. See the [TVMaze API documentation](https://www.tvmaze.com/api) for endpoint details.

The report is an aid to identifying possible gaps. Differences in episode ordering, regional releases, alternate/DVD episode lists, metadata quality, or show matching can affect results. Review the relevant Plex and TVMaze records before making library changes.

## Cache

The script creates `tvmaze_cache.json` in the same directory as the PowerShell script. It stores:

- Plex external-ID/title mappings to TVMaze show IDs.
- TVMaze episode lists and the time each list was cached.

The cache is created automatically and should not be committed to a public repository. It can be deleted while the script is stopped if you want to remove the saved mappings and episode data; the script will rebuild it on the next run. Ensure the script directory is writable by the account running the script.

`-CacheRetentionDays` controls reuse of cached episode lists. The show-ID mapping cache does not currently have a time-based expiry. `-ForceRefresh` refreshes episode lists but does not discard show-ID mappings.

## Troubleshooting

### Plex server URL is required

Supply `-PlexServer`, including the protocol and port. For example:

```powershell
-PlexServer "http://192.168.1.10:32400"
```

### Plex authentication fails

Check the Plex credentials or token, and confirm that the token is still valid. If using username/password authentication, make sure the account can sign in through Plex.tv. If the server restricts unauthenticated connections by IP, check the server's **List of IP addresses and networks that are allowed without auth** setting and the network from which the script is connecting.

### Plex library sections cannot be retrieved

Confirm that the Plex server is online, the URL and port are correct, and the supplied token belongs to an account with access to the TV library. Check any firewall rules between the computer running the script and Plex.

### A show is skipped because it cannot be matched

Check that the Plex show has a correct TVDB or IMDb identifier and that the show exists in TVMaze. If the script has no usable external identifier, its title fallback deliberately avoids ambiguous matches. Correcting the show's metadata in Plex may allow it to be matched on the next run.

### TVMaze requests fail

Confirm that the computer has internet access and can reach `https://api.tvmaze.com`. TVMaze may return temporary errors or rate-limit responses. Try again later if the API is temporarily unavailable.

### Cache appears stale

Run with `-ForceRefresh` to fetch episode lists again, or delete `tvmaze_cache.json` to rebuild both the episode and show-mapping caches.

## Security note

When `-PlexServer` uses HTTPS, the current script installs a certificate policy that accepts all HTTPS certificates in the PowerShell process. This bypasses normal certificate validation and can expose HTTPS traffic to interception if the connection is not trusted. Use this script only on a network you trust. For a safer deployment, remove or replace that certificate-validation override and configure a valid certificate for Plex before relying on HTTPS.

Also, do not store Plex credentials, authentication tokens, or the generated cache file in a public Git repository.

## API, attribution, and data licensing

- [TVMaze public REST API](https://www.tvmaze.com/api)
- [TVMaze API rate limiting and caching information](https://www.tvmaze.com/api#rate-limiting)
- [Creative Commons Attribution-ShareAlike (CC BY-SA)](https://creativecommons.org/licenses/by-sa/4.0/)

The script includes a short delay between TVMaze requests to reduce the likelihood of hitting API rate limits. TVMaze's public API is provided without an API key. TVMaze states that API data is licensed under **CC BY-SA**: users must credit TVMaze as the source and comply with the ShareAlike requirement. This README links to TVMaze as the data source; retain an appropriate attribution when redistributing or adapting data obtained through the API.

The licence you choose for this PowerShell script is separate from the licence that applies to TVMaze data. Review the [TVMaze API licensing terms](https://www.tvmaze.com/api#licensing) before publishing or redistributing TVMaze-derived data.

## Contributing and issues

Bug reports, corrections, and improvements are welcome. When reporting an issue, include the PowerShell version, the command used (with credentials and tokens removed), relevant error output, and whether the affected show has a TVDB or IMDb ID in Plex.
