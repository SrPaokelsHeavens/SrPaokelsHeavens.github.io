---
# --- THE SOVEREIGN CONTROL MASTER MANIFEST v4 ---
# "The Leader commands, the Scriptorium obeys."

# 1. TEMPORAL BOUNDARIES (Retention)
# -----------------------------------------------------------------------------
chapterDaysOnDisplay: 0   # 0 = Chapters never expire from the carousel
newsDaysOnDisplay: 0      # 0 = News never expires from the carousel
autoCleanup: false        # Set to true to physically delete old files from /carousel/

# 2. DENSITY & ORCHESTRATION
# -----------------------------------------------------------------------------
maxTotalCards: 15         # Maximum total cards displayed
chaptersPerNovel: 1       # Show only the last X chapters of each novel
newsLimit: 4              # 0 = No limit for news (filled until maxTotalCards)
displayType: "all"        # "all", "news", or "chapters"

# 3. PRIORITY & WEIGHTING
# -----------------------------------------------------------------------------
typePriority:             # Content appearing first in the logic
  - "chapter"
  - "news"
  - "manual"
featuredFirst: true       # Pull cards with 'featured: true' to the start
priorityList: []          # List of sourceIds to manually pin at the front

# 4. BATCHING LOGIC
# -----------------------------------------------------------------------------
groupingBatch: true       # Combine chapters of the same novel into one card
batchTitleFormat: "{novel}: {count} New Chapters"

# 5. FILTERS
# -----------------------------------------------------------------------------
novelBlacklist: []        # Novel slugs to exclude from auto-generation
newsCategoryFilter:       # Categories allowed to generate news cards
  - "System"
  - "Novel"
  - "Event"
  - "Community"

# 6. MAINTENANCE
# -----------------------------------------------------------------------------
forceRebuild: false       # Set to true to ignore Registry and recreate all cards
purgeOrphaned: true       # Remove Registry entries if original source is deleted

# 7. DAO HIGHLIGHTS
# -----------------------------------------------------------------------------
daoHighlights:
  weeklyContributors:
    limit: 1
    fallbackSourceId: "news-grand-opening"
  celestialTiers:
    limit: 1
    fallbackSourceId: "news-grand-opening"
  sectImmortalMembers:
    limit: 1
    fallbackSourceId: "news-grand-opening"
  immortalSpirits:
    limit: 1
    fallbackSourceId: "news-grand-opening"
  fallbackCard:
    id: "news-grand-opening"
    title: "Grand Opening"
    excerpt: "The pavilion just opened its doors. The first Dao highlights will appear as soon as new offerings arrive."
    image: "https://images.unsplash.com/photo-1489515217757-5fd1be406fef?q=80&w=1200"
    url: "/news"
    publishDate: "2026-02-01"
    type: "manual"
    category: "System"
    featured: false
    priority: 0
---

# SYSTEM LOG: SOVEREIGN
- Created: 2026-02-08
- Current Status: Awaiting The Forge
