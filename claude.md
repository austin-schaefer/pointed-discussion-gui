# Pointed Discussion - MTG Card Comments Archive

## Project Overview

This is a static site generator for browsing historical Magic: The Gathering card comments from the original Gatherer website. The site preserves and makes accessible thousands of community comments from MTG's digital history.

## Data Structure

### Directory Layout
```
/data/
  /199x/          # 1990s sets
    /1993/
      1993-08-05 LEA.json
    /1994/
      ...
  /200x/          # 2000s sets
  /201x/          # 2010s sets
```

### JSON Format
Each JSON file represents a set, with cards as keys and comment arrays as values:
```json
{
  "100: Control Magic": [
    {
      "author": "username",
      "author_id": 314,
      "datetime": "2009-07-01 11:06:26",
      "id": 10612,
      "text_parsed": "HTML version with links",
      "text_posted": "Plain text version",
      "timestamp": "1246471586870",
      "vote_count": 5,
      "vote_sum": 23
    }
  ]
}
```

## Current Features

### Implemented ✓
- **Card Pages**: Individual pages for each card printing showing all comments
- **Comment Sorting**: Sort by rating, date (newest/oldest)
- **Star Ratings**: Calculate and display 5-star ratings from vote data
- **Search/Index Page**: Browse cards alphabetically with statistics
- **Scryfall Integration**: Fetch card metadata and images (including oracle_id)
- **Card Linking**: Automatic linking between cards in comments
- **Static Files**: CSS and JavaScript for interactivity
- **Sitemap Generation**: For SEO and navigation (includes combined pages)
- **Image Handling**: Download and display card artwork
- **Other Printings Banner**: Compact top banner showing all printings of a card
- **Combined View Pages**: Unified pages showing all comments from all printings
- **Printing Navigation**: Easy navigation between different printings
- **Interactive Filtering**: Filter comments by specific printing on combined pages
- **Oracle ID Grouping**: Cards grouped by oracle_id to link printings together

## Architecture

### Core Components

**Models** (`models.py`)
- `Comment`: Individual comment with author, date, votes, text
- `Card`: Card printing with multiverse_id, name, comments, and Scryfall metadata

**Site Generator** (`sitegenerator.py`)
- Loads JSON data from `/data/` hierarchy
- Generates HTML pages using Jinja2 templates
- Processes card links in comments
- Copies images and static files
- Creates search/index page with statistics

**Data Utilities** (`data_utils.py`)
- Iterator for card entries across all JSON files
- Scryfall data caching
- Card name mapping for link resolution

**Templates**
- `card.html`: Individual card page with comment section
- `search.html`: Main index/search page with alphabetical listing

### Technology Stack
- **Python 3.12+**: Core language
- **Jinja2**: HTML templating
- **Scrython**: Scryfall API client
- **Pillow**: Image processing
- **Requests**: HTTP requests

## Implementation Plan

### ✅ Phase 1: Oracle ID Integration (COMPLETED)
**Goal**: Group printings by card identity

Completed tasks:
1. ✅ Updated Scryfall data fetcher to include `oracle_id`
2. ✅ Added `oracle_id` field to Card model
3. ✅ Created card grouping utilities based on oracle_id
4. ✅ Built index of cards by oracle_id in SiteGenerator

**Key learnings:**
- Scrython uses properties, not methods (use `card.oracle_id` not `card.oracle_id()`)
- Set code is accessed via `card.set` not `card.set_code`
- API updates need rate limiting (~9 req/sec for Scryfall)

### ✅ Phase 2: Other Printings Feature (COMPLETED)
**Goal**: Show related printings on each card page

Completed tasks:
1. ✅ Added "Other Printings" banner to card.html template
2. ✅ Display printings with set info, release date, comment count
3. ✅ Easy navigation between printings via clickable badges
4. ✅ Printings sorted by release date

**Design decisions:**
- Chose compact banner at top instead of column layout for better usability
- Current printing highlighted in blue, others as white badges
- Show comment count on each printing badge
- Responsive design: banner stacks vertically on mobile

### ✅ Phase 3: Combined View Page (COMPLETED)
**Goal**: Create unified view of all comments for a card

Completed tasks:
1. ✅ Created new template: `card_combined.html`
2. ✅ Aggregate all comments from all printings
3. ✅ Added printing badges to each comment (set code)
4. ✅ Implemented filtering by printing with dropdown
5. ✅ Updated sitemap to include combined pages (priority 0.9)
6. ✅ Link from individual printings to combined view

**Features:**
- Dropdown filter to view all printings or specific printing
- Sort by: Highest Rated, Date (Newest/Oldest), By Printing (Release Order)
- Set badges positioned in top-right on desktop, inline on mobile
- JavaScript-powered filtering and sorting (no page reload)
- Generated 210 combined pages from 500 updated cards

### Phase 4: UI/UX Enhancements
**Goal**: Polish the browsing experience

Remaining tasks:
1. Add set symbols/icons where available
2. Improve mobile responsiveness (partially done)
3. ~~Add printing selector dropdown on card pages~~ (completed as combined view)
4. Enhance search with printing differentiation
5. Add "View all printings" link to search results

### Phase 5: Performance & Polish
**Goal**: Optimize and finalize

Remaining tasks:
1. Optimize image loading (lazy loading, WebP)
2. Add progress indicators for site generation
3. Improve error handling for missing Scryfall data
4. Add metadata for social sharing (Open Graph tags)
5. Test with full dataset
6. Update all 24,039 cards with oracle_id (currently 500)

## Development Commands

### Setup
```bash
# Install dependencies
uv sync

# Install development tools
uv sync --group dev
```

### Data Preparation
```bash
# Fetch Scryfall metadata for all cards
fetch-scryfall-data

# Generate card name mapping
generate-cardmap

# Download card images
download-mtg-images
```

### Site Generation
```bash
# Generate full site
pointed-discussion generate-all

# Generate single card (for testing)
pointed-discussion generate-card <multiverse_id>
```

## File Structure

```
src/pointed_discussion/
├── models.py              # Data models (Card, Comment)
├── sitegenerator.py       # Main site generation logic
├── data_utils.py          # Data loading utilities
├── api_utils.py           # Scryfall API helpers
├── file_utils.py          # File system utilities
├── cli.py                 # Command-line interface
├── fetch_scryfall_data.py # Scryfall data fetcher
├── generate_cardmap.py    # Card name mapping generator
├── image_downloader.py    # Card image downloader
├── templates/
│   ├── card.html          # Individual card page
│   ├── search.html        # Main search/index page
│   └── card_combined.html # (TODO) Combined printings view
└── static/
    ├── style.css          # Main stylesheet
    ├── search.css         # Search page styles
    └── search.js          # Search functionality
```

## Key Design Decisions

### Static Site Generation
- Chose static generation over dynamic server for simplicity and GitHub Pages compatibility
- All pages pre-generated, no runtime dependencies
- Fast loading, easy hosting

### Scryfall Integration
- Cache Scryfall data locally to minimize API calls
- Use multiverse_id as primary identifier (matches Gatherer)
- Use oracle_id to group different printings of the same card
- Enrich cards with set info, release dates, artist names
- Rate limiting: 9 requests/second (conservative, Scryfall allows ~10)

### Comment Ratings
- Convert vote_sum and vote_count to 5-star scale
- Formula: `star_rating = vote_sum / (2 * vote_count)`
- Display as Unicode stars with numeric rating

### Card Linking
- Parse `[autocard]Card Name[/autocard]` syntax from comments
- Convert to local links between card pages
- Fall back to plain text if card not in dataset

### Other Printings UI
- **Compact banner design**: Subtle top banner instead of dominant column
- **Current printing highlighted**: Blue badge for current, white for others
- **Comment counts visible**: Shows engagement level for each printing
- **Responsive layout**: Stacks vertically on mobile devices
- **Single click navigation**: Direct links to other printings

### Combined View Architecture
- **Client-side filtering**: JavaScript-powered, no page reload needed
- **Flexible sorting**: By rating, date, or printing release order
- **Visual set badges**: Each comment tagged with set code
- **Dropdown-based filtering**: Clean UI for selecting specific printings
- **Proper CSS hierarchy**: Inherits from main stylesheet, custom additions for combined features

## Future Enhancements

### Short Term
- Complete oracle_id update for all 24,039 cards (currently 500)
- Add search autocomplete with printing disambiguation
- Implement comment permalinks
- Add "jump to comment" functionality
- Export comments to JSON for specific cards
- Add set symbols/icons to printing badges

### Medium Term
- Improve mobile responsiveness further
- Add metadata for social sharing (Open Graph tags)
- Implement image lazy loading optimization
- Add progress indicators during site generation
- Better error handling for missing Scryfall data

### Long Term
- User statistics (most active commenters)
- Comment sentiment analysis
- Timeline view of comments over the years
- Card discussions across printings comparison view
- Integration with other MTG data sources
- Full-text search across all comments

## Testing

### Current Tests
- Basic data loading tests
- Model validation

### Needed Tests
- Template rendering
- Card link processing
- Scryfall integration
- Site generation end-to-end
- Oracle ID grouping logic
- Combined page filtering/sorting JavaScript
- Banner layout responsive behavior

### Manual Testing Completed
- ✅ Other Printings banner displays correctly
- ✅ Navigation between printings works
- ✅ Combined view filtering by printing
- ✅ Combined view sorting (all 4 modes)
- ✅ Responsive layout on mobile (banner, combined view)
- ✅ Set badges display correctly on comments
- ✅ CSS styling consistent across pages

## Deployment

### GitHub Pages
1. Generate static site: `pointed-discussion generate-all`
2. Output goes to `output/` directory
3. Push `output/` contents to `gh-pages` branch
4. Configure custom domain if desired

### Alternative Hosting
- Any static file host works (Netlify, Vercel, S3)
- No server-side processing required
- CDN recommended for images

## Contributing Guidelines

### Code Style
- Follow Python PEP 8 conventions
- Use type hints where applicable
- Use Ruff for linting: `ruff check .`

### Commit Messages
- Descriptive commits with context
- Reference issue numbers where applicable

### Pull Requests
- Update tests for new features
- Update this documentation
- Test with subset of data before full generation

## Resources

- [Scryfall API Documentation](https://scryfall.com/docs/api)
- [Scrython Library Docs](https://scrython.readthedocs.io/)
- [Jinja2 Documentation](https://jinja.palletsprojects.com/)
- [Original Gatherer Archive Credit](https://www.reddit.com/user/cardologist)

## License

Data credit: Original comments from Wizards of the Coast's Gatherer, archived by Reddit user `cardologist`.
