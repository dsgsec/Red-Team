Introduction aux moteurs de modèles
===================================

* * * * *

Les moteurs de modèles lisent les chaînes tokenisées à partir de documents modèles et produisent des chaînes rendues avec des valeurs réelles dans le document de sortie. Les modèles sont couramment utilisés comme format intermédiaire par les développeurs Web pour créer un contenu de site Web dynamique. L'injection de modèle côté serveur ( `SSTI`) consiste essentiellement à injecter des directives de modèle malveillantes dans un modèle, en tirant parti des moteurs de modèle qui mélangent de manière non sécurisée l'entrée de l'utilisateur avec un modèle donné.

Vous trouverez ci-dessous quelques applications que vous pouvez exécuter localement pour mieux comprendre les modèles. Si vous ne parvenez pas à le faire, ne vous inquiétez pas. Les sections suivantes présentent des exercices avec diverses applications utilisant des modèles.

Considérons maintenant les documents suivants :

#### app.py

Code : Python

```
#/usr/bin/python3
from flask import *

app = Flask(__name__, template_folder="./")

@app.route("/")
def index():
	title = "Index Page"
	content = "Some content"
	return render_template("index.html", title=title, content=content)

if __name__ == "__main__":
	app.run(host="127.0.0.1", port=5000)

```

#### index.html

Code : html

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>{{title}}</h1>
    <p>{{content}}</p>
</body>
</html>

```

Lorsque nous visitons le site Web, nous recevons une page HTML contenant les valeurs des variables `title`et `content`évaluées à l'intérieur des doubles parenthèses sur la page de modèle. Assez simple, et comme nous pouvons le voir, l'utilisateur n'a aucun contrôle sur les variables. Que se passe-t-il lorsque l'entrée de l'utilisateur entre dans un modèle sans aucune validation ?

#### app.py

Code : Python

```
#/usr/bin/python3
from flask import *

app = Flask(__name__, template_folder="./")

@app.route("/")
def index():
	title = "Index Page"
	content = "Some content"
	return render_template("index.html", title=title, content=content)

@app.route("/hello", methods=['GET'])
def hello():
	name = request.args.get("name")
	if name == None:
		return redirect(f'{url_for("hello")}?name=guest')
	htmldoc = f"""
	<html>
	<body>
	<h1>Hello</h1>
	<a>Nice to see you {name}</a>
	</body>
	</html>
	"""
	return render_template_string(htmldoc)

if __name__ == "__main__":
	app.run(host="127.0.0.1", port=5000)

```

Dans ce cas, nous pouvons injecter directement une expression de modèle et le serveur l'évaluera. Il s'agit d'un problème de sécurité pouvant entraîner l'exécution de code à distance sur l'application cible, comme nous le verrons dans les sections suivantes.

#### cURL - Interagir avec la cible

  cURL - Interagir avec la cible

```
dsgsec@htb[/htb]$ curl -gis 'http://127.0.0.1:5000/hello?name={{7*7}}'

HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 79
Server: Werkzeug/2.0.2 Python/3.9.7
Date: Mon, 25 Oct 2021 00:12:40 GMT

	<html>
	<body>
	<h1>Hello</h1>
	<a>Nice to see you 49</a> # <-- Expresion evaluated
	</body>
	</html>

```

