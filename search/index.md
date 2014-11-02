---
layout: null
---
<style>
#search-demo-container {
  max-width: 40em;
padding: 1em;
margin: 1em auto;
border: 1px solid lightgrey;
}
#search-input {
display: inline-block;
padding: .5em;
width: 100%;
       font-size: 0.8em;
       -webkit-box-sizing: border-box;
       -moz-box-sizing: border-box;
       box-sizing: border-box;
}
</style>
<div id="search-demo-container">
  <input type="text" id="search-input" placeholder="search...">
  <ul id="results-container"></ul>
</div>

<script src="../search/jekyll-search.js"></script>
<script type="text/javascript">
  SimpleJekyllSearch.init({
      searchInput: document.getElementById('search-input'),
      resultsContainer: document.getElementById('results-container'),
      dataSource: '../search.json',
      searchResultTemplate: '<li><a href="{url}" title="{desc}">{title}</a></li>',
      noResultsText: 'No results found',
      limit: 10,
      fuzzy: true,
  })
</script>
