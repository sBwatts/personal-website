
library(reticulate)

# Source the Python file
source_python("open-alex.py")

# The classes are now available directly
syncer <- FilteredOpenAlexSync("research/articles")

syncer$sync_author_works(
  orcid = "0000-0002-5108-9055",
  limit = 30L  # Note: use 20L (integer) not just 20
)