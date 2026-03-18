# Watchtower

Watchtower is a CLI scraper for bank information. It opens bank or lender pages in Chrome, collects product and TIN content, extracts cleaned HTML and text, and writes the results to local files.

## What it does

- Loads organization definitions from `settings.json`
- Starts a browser session with `pydoll-python`
- Scrapes product pages for one or more configured organizations
- Extracts normalized `html` and `txt` output with `trafilatura`
- Saves output under `data/<organization-name>/`

## Prerequisites

- Python `3.13+`
- [`uv`](https://docs.astral.sh/uv/)
- Google Chrome installed locally

## Installation

```sh
uv sync
```

## Configuration

Runtime settings come from:

- `pyproject.toml`: project metadata shown by the CLI
- `settings.json`: worker, logging, and organization definitions

`settings.json` is expected to look like this:

```json
{
  "worker": { "max_num": 8 },
  "logging": {
    "level": "info",
    "time_format": "%Y-%m-%d %H:%M:%S",
    "utc": false
  },
  "orgs": [
    {
      "name": "example_bank",
      "base_url": "https://bank-website",
      "product_list_url_path": "/path/to/product/list",
      "product_info_prune_xpaths": [],
      "tin_url": "https://bank-website/tin",
      "with_captcha": false,
      "with_sleep": false,
      "product_list_action": null,
      "product_list_query": "a.product-link",
      "product_list_tab_query": ".tab-selector"
    }
  ]
}
```

Organization names in `orgs[].name` become the accepted CLI values for `watchtower scrape`.

## Usage

Show available commands:

```sh
uv run watchtower --help
```

Scrape one organization:

```sh
uv run watchtower scrape example_bank
```

Scrape multiple organizations:

```sh
uv run watchtower scrape example_bank another_bank
```

Scrape every configured organization:

```sh
uv run watchtower scrape --all
```

## Output

For each organization, Watchtower writes files to:

```text
data/<organization-name>/
```

Generated files include:

- `product_<slug>.html`
- `product_<slug>.txt`
- `tin.html`
- `tin.txt`

## Notes

- Product URLs are discovered from the configured product list page.
- `product_list_action` is optional. Set it to `"click"` for sites where links must be opened by interaction; leave it `null` to read URLs from `href`.
- `product_list_tab_query` is optional and supports list pages split across tabs.
- `with_captcha` enables Cloudflare bypass handling for navigation.
- `with_sleep` adds a fixed delay after navigation for slower sites.
