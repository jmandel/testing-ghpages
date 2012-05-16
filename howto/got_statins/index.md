---
layout: page
title: Got Statins? App
tagline: fourth steps
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>


##Developers Documentation: GotStatins - SMART Connect in Action

You should read [HOWTO Build a SMART App](../../build_a_smart_app) first.

Here, we build a complete SMART Connect app from pure HTML and Javascript. Modeled on the uniquely popular, laser-focused chef-d'oeuvre [isitchristmas.com](http://isitchristmas.com/), this app obtains a patient's medication list, determines whether there are statins on board, and relays the answer in large, bold typeface. Using the SMART Connect API, there's no need for explicit authentication, session-management, or AJAX calls. 

Every SMART app exposes a URL at: 

<ol><li>
index.html: loads smart-api-client.js to provide the app's functionality.</li></ol>

index.html is where all the fun happens! First, don't forget to include 

{% highlight html %}
<script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
{% endhighlight  %}

Inside index.html, we invoke SMART.ready() with a callback function so we can be sure that the SMART client-side library has finished loading. At this point, a SMART JavaScript object that provides some basic context (such as SMART.user, which provides the name and ID of the user who's running the app, and SMART.record which provides the name and ID of the patient whose record is loaded).

And now we can fetch some data from the patient record: 

{% highlight html %}
SMART.MEDICATIONS_get().success(function(meds) { ... });
{% endhighlight  %}

As you may remember from the HOWTO, meds is a SMART Response object that includes an RDF graph of medications (type sp:Medication). These medications include fields such as the [RxNorm](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_RDF_Data) concept ID, as well as fulfillment history when available. The whole RDF graph is wrapped up in an [rdfquery](http://code.google.com/p/rdfquery/) object, providing convenience query functions such as .where(), .optional(), and .filter() which we can use to interact with the graph. 

In this demo app, we use a very low-tech way to figure out of a medication is a station: find its dcterms:title, and string match against the names of all known statins. 

{% highlight html %}
var medlist = meds.graph
                     .where("?m rdf:type sp:Medication")
                     .where("?m sp:drugName ?n")
                     .where("?n dcterms:title ?drugname");
{% endhighlight  %}

Here's how it all comes together: 


{% highlight html %}
<!DOCTYPE html>
<html>
<head>
<title>Got Statins?</title>
</head>
<body>

<h1 style="font-family: Arial, sans-serif;">Got Statins?</h1>
<a id="TheAnswer">
...
</a>
<script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
<script>

SMART.ready(function(){

    SMART.MEDICATIONS_get().success(function(meds) {

        var medlist = meds.graph
        .where("?m rdf:type sp:Medication")
        .where("?m sp:drugName ?dn")
        .where("?dn dcterms:title ?drugname");
        var answer = false;

        for (var i = 0; i < medlist.length; i++) {
            console.log(medlist[i].drugname.value);
            if (is_a_statin(medlist[i].drugname.value))
                answer = true;
        }

        document.getElementById("TheAnswer").innerHTML = answer ? "Yes." : "No.";

    });

    var is_a_statin = function(drug) {
        if (drug.match(/statin/i)) return true;
        if (drug.match(/Advicor/i)) return true;
        if (drug.match(/Altoprev/i)) return true;
        if (drug.match(/Caduet/i)) return true;
        if (drug.match(/Crestor/i)) return true;
        if (drug.match(/Lescol/i)) return true;
        if (drug.match(/Lipitor/i)) return true;
        if (drug.match(/Mevacor/i)) return true;
        if (drug.match(/Pravachol/i)) return true;
        if (drug.match(/Simcor/i)) return true;
        if (drug.match(/Vytorin/i)) return true;
        if (drug.match(/Zocor/i)) return true;

        return false;
    }
});
</script>
</body>
</html>
{% endhighlight  %}

This little app can be statically hosted anywhere. If a SMART container loads it up in an iframe, it will have instant access to the in-context patient record and give us a quick yes or no. 
