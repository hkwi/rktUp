---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: home
---
<script src="https://cdn.jsdelivr.net/npm/vue" defer></script>
<script type="text/javascript" defer>
// https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries
// https://query.wikidata.org/#SELECT%20%3Fo%20%3Fs%20%3FsLabel%20%3FpLabel%20WHERE%20%7B%0A%20%20%3Fs%20wdt%3AP619%20%3Fo%20.%0A%20OPTIONAL%7B%0A%20%20%20%3Fs%20wdt%3AP1427%20%3Fp%20.%0A%20%7D%0A%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22ja%2Cen%22.%20%7D%0A%7D%0AORDER%20BY%20DESC%28%3Fo%29%0ALIMIT%2010

// NASA
//  https://www.nasa.gov/launchschedule/
//  results : https://www.nasa.gov/subject/7451/launches/
// JAXA
//  http://fanfun.jaxa.jp/faq/detail/290.html

// http://negi-magnet.hatenablog.com/entry/2014/01/13/132406

var dtend = new Date();
dtend.setMonth(dtend.getMonth() + 2);

var query = `
SELECT ?o ?s ?sLabel ?name ?opLabel ?ctLabel ?pLabel WHERE {
  ?s wdt:P619 ?o .
 OPTIONAL{ ?s wdt:P1427 ?p . }
 OPTIONAL{ ?s wdt:P137 ?op . }
 OPTIONAL{ ?s wdt:P1079 ?ct . }
 FILTER (?o < "${ dtend.toISOString() }"^^xsd:dateTime )
 SERVICE wikibase:label { bd:serviceParam wikibase:language "${navigator.language},en". }
}
ORDER BY DESC(?o)
LIMIT 10
`;
fetch("https://query.wikidata.org/sparql?query="+encodeURIComponent(query), {
	headers: {
		accept: 'application/json'
	}
}).then(
	resp => {
		if(resp.ok){
			resp.json().then(
				data => {
					var labels = data.results.bindings.map(r=>{
						var d = {
							name: r.sLabel.value,
							url: r.s.value,
						};
						try{
							d.date = new Date(Date.parse(r.o.value)).toLocaleDateString();
						}catch(e){
							console.log(e);
							d.date = r.o.value;
						}
						if (r.pLabel){
							d.loc = r.pLabel.value;
						}
						if (r.opLabel){
							d.op = r.opLabel.value;
						} else if (r.ctLabel){
							d.op = r.ctLabel.value;
						}
						return d;
					})
					app = new Vue({
						el: "#launches",
						data: {
							results: labels
						}
					})
				}
			);
		}
	}
);
</script>

Launches within two month. ([Wikidata Query](https://query.wikidata.org/#SELECT%20%3Fo%20%3Fs%20%3FsLabel%20%3FpLabel%20WHERE%20%7B%0A%20%20%3Fs%20wdt%3AP619%20%3Fo%20.%0A%20OPTIONAL%7B%0A%20%20%20%3Fs%20wdt%3AP1427%20%3Fp%20.%0A%20%7D%0A%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_SELECT%5D%2Cen%22.%20%7D%0A%7D%0AORDER%20BY%20DESC%28%3Fo%29%0A))
<table id="launches">
 <tr><th>Date</th><th>Name</th><th>Operator</th><th>Location</th></tr>
 {% raw %}
 <tr v-for="r in results">
   <td>{{ r.date }}</td>
   <td><a v-bind:href="r.url">{{ r.name }}</a></td>
   <td>{{ r.op }}</td>
   <td>{{ r.loc }}</td>
 </tr>
 {% endraw %}
</table>

