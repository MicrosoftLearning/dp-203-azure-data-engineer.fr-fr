---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

# Exercices Azure Ingénieurs Données ing

Les exercices suivants prennent en charge les modules de formation sur Microsoft Learn qui prennent en charge la [certification Microsoft Certified : Azure Ingénieurs Données Associate](https://learn.microsoft.com/certifications/azure-data-engineer/).

Pour effectuer ces exercices, vous avez besoin d’un abonnement Microsoft Azure dans lequel vous disposez d’autorisations d’administration. Pour certains exercices, vous devrez peut-être également accéder à un [locataire](https://learn.microsoft.com/power-bi/fundamentals/service-self-service-signup-for-power-bi) Microsoft Power BI.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Exercice | En ILT, c’est un... |
| --- | --- |
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) | {{ activity.lab.ilt-use }} |
{% endfor %}
