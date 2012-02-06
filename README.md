<H1>lazy-proxy</H1>
lazy is meant to be a very simple proxy route for node.js, in particular to be used with express as middleware.  The idea
is to make it easy to add multiple outbound routes by adding "/abbreviation/json/and-then-my-path-here" or "/abbreviation/view/page/and-then-my-path-here" - where abbreviation could be say "fb" to mean "Facebook" and then  "json" would return the raw JSON (for use with client side AJAX calls) and "view" would then also append server-side view to have the data rendered on to "page".

So an express based route:

```
app.all('/:label/:mode/*',
  ensureAuthenticated,
  function(req, res) {
    console.log(req.session);
    
    //forcedotcom
    if(req.session["forcedotcom"] && req.params.label == "fdc") {
      var restOptions = {
        useHTTPS : true,
        host : req.session["forcedotcom"].instance_url.replace('https://',''),
        headers: {
            'Authorization': 'OAuth '+req.session["forcedotcom"].access_token,
            'Accept':'application/jsonrequest',
            'Cache-Control':'no-cache,no-store,must-revalidate'
          }
      }

      lazyproxy.send(restOptions,req,res);
    }

    //facebook
    if(req.session["facebook"] && req.params.label == "fb") {
      var restOptions = {
        useHTTPS : true,
        host : 'graph.facebook.com',
        headers: {
            'Authorization': 'OAuth '+req.session["passport"]["user"].access_token,
            'Accept':'application/jsonrequest',
            'Cache-Control':'no-cache,no-store,must-revalidate'
          }
      }

       lazyproxy.send(restOptions,req,res);
    }

  });
  ```

  Would allow an application to forward those routes to Force.com and Facebook.  Where:

  ```
  /fdc/json/services/data/v22.0/query?q=SELECT%20ID%20FROM%20Account
  ```

  Would send the result of SOQL callout directly back as json,

  ```
/fdc/view/account/services/data/v22.0/sobjects/Account/{accountId}
  ```

Would call render("account",{accountdata}) instead.
