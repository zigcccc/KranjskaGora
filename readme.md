# Kranjska gora AMP strani
## Problemi in popravki

### 1. CSS komponente AMP strani
Prepovedana je uporaba pravila @key-frames v znački `<style amp-boilerplate>`. Razlog je v razmeroma počasnem procesiranju podatkov o animaciji na strani. Ni pa popolnoma prepovedana uporaba keyframe-ov. Zanje je rezervirana značka `<style amp-keyframes>`. Obstaja nekaj pravil za to značko:

* mora nastopati kot zadnji element značke `<body>`
* vsebuje lahko le `@keyframes`, `@media` in `@supports` pravila
* ne sme biti večja od 500kb

Pravilo `@keyframes` ima še nadaljne zahteve in sicer glede vrst pravil, ki jih lahko vsebuje. Dovoljena so le pravila, ki so **GPU-accelerated** in sicer:

* opacity
* transform
* -prefix-transform

Torej koda bi morala izgledati tako:

```html
<head>
	<!-- vsebina head značke -->
	<style amp-boilerplate> <!-- lahko tudi amp-custom, zelo majhna razlika med enim in drugim -->
	    body{
	        -webkit-animation:-amp-start 8s steps(1,end) 0s 1 normal both;
	        -moz-animation:-amp-start 8s steps(1,end) 0s 1 normal both;
	        -ms-animation:-amp-start 8s steps(1,end) 0s 1 normal both;
	        animation:-amp-start 8s steps(1,end) 0s 1 normal both
	    }
	</style>
</head>
<body>
	<!-- Vsa vsebina body značke -->
	<style amp-keyframes>
		@-webkit-keyframes -amp-start{from{opacity:0}to{opacity:1}}
		@-moz-keyframes -amp-start{from{opacity:0}to{opacity:1}}
		@-ms-keyframes -amp-start{from{opacity:0}to{opacity:1}
		@-o-keyframes -amp-start{from{opacity:0}to{opacity:1}
		@keyframes -amp-start{from{opacity:0}to{opacity:1}}
	</style>
</body>
```
Prav tako je prepovedana uporaba **inline-style**-ov v samih HTML značkah. Torej ne sme se zgoditi, kot se je zgodilo v tem primeru:

```html
<p style="text-align: justify;">...</p>
```

### 2. Značka `<iframe>`
Značka `iframe` se na strani lahko pojavlja le kot naslednica značke `noscript`. V primeru, da je želimo uporabiti na strani drugje kot v znački noscript, moramo uporabiti privzeto amp značko `amp-iframe`.

Tudi za te značke pa obstaja pravilo glede *above-render-fold* problema, in sicer naj značke ne bi nastopale prej kot:

* 600px od vrha oz.
* v prvih 75% viewporta (karkoli od tega nastopi prej)

Temu "problemu" se lahko izognemo v primeru, da nastavimo **placeholder image**, ki ga zavijemo v značko `<amp-img>`, katera nastopa znotraj značke `<amp-iframe>`.
Primer:

```html
<amp-img layout="fill" src="https://i.vimeocdn.com/video/536538454_640.webp" placeholder></amp-img>
```
Torej na naši strani bi koda morala izgledati nekako tako:

```html
<amp-iframe 
	allowfullscreen="true" 
	allowtransparency="true"
	frameborder="0"
	height="480"
	scrolling="no"
	src="https://www.facebook.com/plugins/video.php?href=https%3A%2F%2Fwww.facebook.com%2Fplanica.si%2Fvideos%2F10154454887761687%2F&amp;show_text=0&amp;width=560"
	style="border:none;overflow:hidden"
	width="100%"
>
<amp-img placeholder layout="fill" src="{{ url naslov do statične naslovne slike }}"></amp-img>
</amp-iframe>
```

### 3. Značke `<form>` na strani
Forme na AMP straneh so sicer dovoljene, ampak pod posebnimi pogoji in z uporabo privzete `amp-form` skripte. Slednja izgleda tako:

```html
<script async custom-element="amp-form" src="https://cdn.ampproject.org/v0/amp-form-0.1.js"></script>
```
V sami strukturi forme se potem ne spremeni veliko, posebnost je atribut značke `<form action="">`, ki se spremeni v `<form action-xhr="">`.

Torej za naš primer (če že moramo uporabiti formo, kar mislim da ne bi bilo potrebno in bi bilo bolje, če je ne bi), bi koda izgledala približno tako:

```html
<head>
	<!-- ostala vsebina head značke -->
	<script async custom-element="amp-form" src="https://cdn.ampproject.org/v0/amp-form-0.1.js"></script>
</head>
<body>
	<form 
		action-xhr="/si/aktualno/809-FIS-finale-svetovnega-pokala-v-smucarskih-skokih-Planica-2017"
		id="aspnetForm" 
		method="post" 
		name="aspnetForm">
	   <p><strong>Kdaj: </strong>22.3. - 25.3.2018</p>
	
	   <p><strong>Kje:</strong> Planica, Letalnica bratov Gori&scaron;ek</p>
	
	   <p><strong>Organizator: </strong>OK Planica 2018</p>
	
	   <p><b>Informacije</b>: <a href="http://www.planica.si/#">planica@sloski.si</a> ali 051 61 09 44</p>
	
	   <p><strong>www:&nbsp;</strong><a href="http://www.planica.si/skoki" target="_blank">http://www.planica.si</a></p>
	</form>
</body>
```
### 4. Google Analytics
Implementacija Google Analytics na AMP straneh je možna, ampak nekoliko drugačna od klasične implementacije. Omejeni smo z možnostjo sledljivih elementov, na voljo so nam:

* pageview
* anchor clicks
* timer
* scrolling
* AMP Carousel changes

V začetni fazi bi za naše potrebe zadostovalo sledenje pageview-em in timer-ju.

**4.1 Implementacija Google Analytics kode**

V `head` značko vstavimo kodo potrebno za delovanje značke `amp-analytics` in sicer:

```html
<script async custom-element="amp-analytics" src="https://cdn.ampproject.org/v0/amp-analytics-0.1.js"></script>
```
**4.2 Implementacija sledljivih parametrov**

Pri koncu značke `body` vstavimo kodo, kjer podamo podatke našega analytics računa in navedemo sledljive elemente. To izgleda tako:

```html
<amp-analytics>
  <script type="application/json">
  {
    "requests": {
      "event": "https://amp-publisher-samples-staging.herokuapp.com/amp-analytics/ping?user=u5t3l0xo&account=ampbyexample&event=${eventId}"
    },
    "triggers": {
      "pageTimer": {
        "on": "timer",
        "timerSpec": {
          "interval": 10,
          "maxTimerLength": 600
        }
      },
      "trackPageview": {
        "on": "visible",
        "request": "event",
        "vars": {
          "eventId": "pageview"
        }
      }
    }
  }
  </script>
</amp-analytics>
```
V polju "event" moramo seveda vnesti podatke našega Analytics računa, ostale parametre nastavljamo glede na potrebe (npr. maxTimerLength je povsem poljuben).

Uradna dokumentacija za amp-analytics je na voljo [na tem naslovu](https://www.ampproject.org/docs/reference/components/amp-analytics).