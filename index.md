---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

# Exercices Azure Data Engineering

Les exercices suivants prennent en charge les modules de formation sur Microsoft Learn qui prennent en charge la certification [Microsoft Certified: Azure Data Engineer Associate](https://learn.microsoft.com/certifications/azure-data-engineer/).

Pour effectuer ces exercices, vous avez besoin d’un [abonnement Microsoft Azure](https://azure.microsoft.com/free) dans lequel vous disposez d’un accès administratif. Pour certains exercices, vous devrez peut-être également accéder à un [locataire Microsoft Power BI](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Exercice | Dans une ILT, il s’agit de... |
| --- | --- |
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) | {{ activity.lab.ilt-use }} |
{% endfor %}
