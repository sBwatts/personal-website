import requests
from pathlib import Path
import yaml
from datetime import datetime
import time
import re

class OpenAlexArticleSync:
    def __init__(self, articles_dir="research/articles"):
        self.base_url = "https://api.openalex.org"
        self.articles_dir = Path(articles_dir)
        self.articles_dir.mkdir(parents=True, exist_ok=True)
        
    def fetch_author_works(self, author_name=None, orcid=None, limit=50):
        """Fetch works by author from OpenAlex"""
        if orcid:
            url = f"{self.base_url}/works"
            params = {
                'filter': f'author.orcid:{orcid}',
                'per-page': limit,
                'sort': 'publication_date:desc',
                'mailto': 'sbwatts@txstate.edu'
            }
        elif author_name:
            url = f"{self.base_url}/works"
            params = {
                'filter': f'author.search:{author_name}',
                'per-page': limit,
                'sort': 'publication_date:desc',
                'mailto': 'sbwatts@txstate.edu'
            }
        else:
            raise ValueError("Must provide either author_name or orcid")
        
        print(f"Fetching works from OpenAlex...")
        response = requests.get(url, params=params)
        response.raise_for_status()
        
        data = response.json()
        works = data.get('results', [])
        
        print(f"Found {len(works)} works")
        return self._parse_works(works)
    
    def fetch_by_doi(self, doi):
        """Fetch a specific work by DOI"""
        url = f"{self.base_url}/works/doi:{doi}"
        response = requests.get(url, params={'mailto': 'sbwatts@txstate.edu'})
        response.raise_for_status()
        
        work = response.json()
        return self._parse_works([work])[0]
    
    def _parse_works(self, works):
        """Parse OpenAlex works into article format, preferring published versions"""
        works_by_title = {}
        
        for work in works:
            title = work.get('title', 'Untitled')
            normalized_title = re.sub(r'[^\w\s]', '', title.lower()).strip()
            
            if normalized_title not in works_by_title:
                works_by_title[normalized_title] = []
            works_by_title[normalized_title].append(work)
        
        selected_works = []
        for normalized_title, work_group in works_by_title.items():
            if len(work_group) == 1:
                selected_works.append(work_group[0])
            else:
                print(f"  ⚠ Found {len(work_group)} versions of: {work_group[0].get('title', '')[:50]}...")
                
                best_work = work_group[0]
                for work in work_group:
                    work_type = work.get('type', '')
                    best_type = best_work.get('type', '')
                    
                    if work_type == 'article' and best_type != 'article':
                        best_work = work
                        print(f"    → Selecting journal version")
                    elif work_type == best_type and work.get('doi') and not best_work.get('doi'):
                        best_work = work
                    elif work_type == best_type and work.get('cited_by_count', 0) > best_work.get('cited_by_count', 0):
                        best_work = work
                
                selected_works.append(best_work)
        
        articles = []
        for work in selected_works:
            authors = []
            if work.get('authorships'):
                authors = [a['author']['display_name'] for a in work['authorships'][:3]]
            author_str = ', '.join(authors)
            if len(work.get('authorships', [])) > 3:
                author_str += ' et al.'
            
            pub_date = work.get('publication_date', '')
            if not pub_date:
                pub_year = work.get('publication_year')
                pub_date = f"{pub_year}-01-01" if pub_year else datetime.now().strftime('%Y-%m-%d')
            
            title = work.get('title', 'Untitled')
            slug = self._create_slug(title, work.get('id', '').split('/')[-1])
            
            categories = []
            if work.get('concepts'):
                categories = [c['display_name'] for c in work['concepts'][:5] if c.get('score', 0) > 0.3]
            
            abstract = work.get('abstract', '')
            if not abstract and work.get('abstract_inverted_index'):
                abstract = self._reconstruct_abstract(work['abstract_inverted_index'])
            
            cited_by_count = work.get('cited_by_count', 0)
            if cited_by_count is None:
                cited_by_count = 0
            
            print(f"  Citations for '{title[:40]}...': {cited_by_count}")
            
            article = {
                'slug': slug,
                'title': title,
                'author': author_str,
                'date': pub_date,
                # 'description': abstract[:500] if abstract else 'No abstract available.',
                # 'categories': categories,
                'doi': work.get('doi', '').replace('https://doi.org/', ''),
                'url': work.get('doi', ''),
                'open_access': work.get('open_access', {}).get('is_oa', False),
                'pdf_url': work.get('open_access', {}).get('oa_url', ''),
                'cited_by_count': cited_by_count,
                'publication_venue': work.get('primary_location', {}).get('source', {}).get('display_name', ''),
                'openalex_id': work.get('id', ''),
                'work_type': work.get('type', ''),
            }
            
            articles.append(article)
        
        return articles
    
    def _create_slug(self, title, fallback_id):
        """Create URL-friendly slug from title"""
        slug = title.lower()
        slug = re.sub(r'[^\w\s-]', '', slug)
        slug = re.sub(r'[-\s]+', '-', slug)
        slug = slug[:80]
        return slug if slug else fallback_id
    
    def _reconstruct_abstract(self, inverted_index):
        """Reconstruct abstract from inverted index"""
        if not inverted_index:
            return ''
        
        words_positions = []
        for word, positions in inverted_index.items():
            for pos in positions:
                words_positions.append((pos, word))
        
        words_positions.sort()
        return ' '.join([word for _, word in words_positions])
    
    def create_article_index(self, article):
        """Create index.qmd for an article"""
        slug = article['slug']
        article_path = self.articles_dir / slug
        article_path.mkdir(parents=True, exist_ok=True)
        
        is_preprint = False
        work_type = article.get('work_type', '').lower()
        venue = article.get('publication_venue', '').lower()
        
        if work_type == 'posted-content' or 'crimrxiv' in venue or 'arxiv' in venue or 'preprint' in venue:
            is_preprint = True
        
        yaml_data = {
            'title': article['title'],
            'author': article['author'],
            'date': article['date'],
        }
        
        # if is_preprint:
        #     yaml_data['subtitle'] = '🔬 Preprint'
        
        if article.get('description') and article['description'] != 'No abstract available.':
            yaml_data['description'] = article['description']
        
        # if article.get('categories'):
        #     categories = article['categories'].copy() if article['categories'] else []
        #     if is_preprint and 'Preprint' not in categories:
        #         categories.insert(0, 'Preprint')
        #     yaml_data['categories'] = categories
        # elif is_preprint:
        #     yaml_data['categories'] = ['Preprint']
        
        if article.get('doi'):
            yaml_data['doi'] = article['doi']
        
        if article.get('url'):
            yaml_data['citation-url'] = article['url']
        
        yaml_data['format'] = {
            'html': {
                'toc': True
            }
        }
        
        content_parts = []
        
        if is_preprint:
            content_parts.append("""
::: {.callout-warning}
## 🔬 Preprint
This is a preprint that has not undergone peer review. Findings should be interpreted with caution.
:::
""")
        
        pub_info = []
        if article.get('publication_venue'):
            venue_text = f"**Published in:** {article['publication_venue']}"
            # if is_preprint:
            #     venue_text += " *(Preprint Server)*"
            pub_info.append(venue_text)
        
        citation_count = article.get('cited_by_count', 0)
        if citation_count and citation_count > 0:
            pub_info.append(f"**Citations:** {citation_count}")
        
        if article.get('open_access'):
            pub_info.append(f"**Open Access:** Yes")
        
        if pub_info:
            content_parts.append("## Publication Details\n\n" + '\n\n'.join(pub_info) + "\n")
        
        links = []
        if article.get('url'):
            links.append(f"[DOI Link]({article['url']})")
        if article.get('pdf_url'):
            links.append(f"[PDF]({article['pdf_url']})")
        if article.get('openalex_id'):
            links.append(f"[OpenAlex]({article['openalex_id']})")
        
        if links:
            content_parts.append("## Links\n\n" + ' | '.join(links) + "\n")
        
        content_body = '\n'.join(content_parts)
        
        yaml_str = yaml.dump(
            yaml_data, 
            default_flow_style=False,
            allow_unicode=True,
            sort_keys=False,
            width=float('inf')
        ).strip()
        
        index_content = f"---\n{yaml_str}\n---\n\n{content_body}"
        
        index_file = article_path / 'index.qmd'
        with open(index_file, 'w', encoding='utf-8') as f:
            f.write(index_content)
        
        print(f"✓ Created: {slug}{' 🔬' if is_preprint else ''}")
        return article_path
    
    def sync_author_works(self, author_name=None, orcid=None, limit=50):
        """Sync all works by an author"""
        articles = self.fetch_author_works(author_name=author_name, orcid=orcid, limit=limit)
        
        created = []
        for article in articles:
            try:
                path = self.create_article_index(article)
                created.append(path)
                time.sleep(0.1)
            except Exception as e:
                print(f"✗ Error creating {article.get('slug')}: {e}")
        
        print(f"\n✓ Synced {len(created)}/{len(articles)} articles successfully!")
        return created


class FilteredOpenAlexSync(OpenAlexArticleSync):
    """Extended version with filtering options"""
    
    def fetch_author_works(self, author_name=None, orcid=None, limit=50, 
                          min_citations=None, publication_year_from=None,
                          only_open_access=False, exclude_types=None):
        """Fetch works with advanced filtering"""
        filters = []
        
        if orcid:
            filters.append(f'author.orcid:{orcid}')
        elif author_name:
            filters.append(f'author.search:{author_name}')
        else:
            raise ValueError("Must provide either author_name or orcid")
        
        if min_citations:
            filters.append(f'cited_by_count:>{min_citations}')
        
        if publication_year_from:
            filters.append(f'publication_year:>{publication_year_from}')
        
        if only_open_access:
            filters.append('is_oa:true')
        
        if exclude_types:
            for t in exclude_types:
                filters.append(f'type:!{t}')
        
        url = f"{self.base_url}/works"
        params = {
            'filter': ','.join(filters),
            'per-page': limit,
            'sort': 'publication_date:desc',
            'mailto': 'sbwatts@txstate.edu'
        }
        
        print(f"Fetching works with filters: {filters}")
        response = requests.get(url, params=params)
        response.raise_for_status()
        
        works = response.json().get('results', [])
        print(f"Found {len(works)} works")
        return self._parse_works(works)


if __name__ == "__main__":
    syncer = FilteredOpenAlexSync("research/articles")
    syncer.sync_author_works(
        orcid="0000-0002-5108-9055",
        limit=50
    )
