# Search


<form onsubmit="return search()">
    <input type="text" id="query">
    <button onclick="search()">Search</button>
</form>

<h1>Results:</h2>
<ul id="search-result" class="search-list">
</ul>

<script src="../index.json"></script>
<script>
function search(){
    let query = document.getElementById("query").value;
    let results = articleIndex.filter(article => article.body.match(query));

    let ul = document.getElementById("search-result");
    ul.innerHTML = "";
    
    for(let i = 0; i < results.length; i++) {
        let elem = results[i];

        let li = document.createElement("li");
        li.setAttribute("class", "search-item");

        let elemLink = document.createElement("a");
        elemLink.innerHTML = elem.title;
        elemLink.setAttribute("href", elem.permalink);
        elemLink.setAttribute("class", "search-item-link");

        li.appendChild(elemLink);
        ul.appendChild(li);
    }

    return false;
}
</script>
