---
title: "Raise the edge of the asset tracking"
excerpt: "Fleet management, a study case"
collection: portfolio
---

This is a use case, I worked on, with a Transportation company, leader in the state, with 102 trucks, and 91 drivers to manage.

# Situation  
This project was initiated by the need of tracking in real-time the moving assets. The result, as we see, is a spatial map with multiple layers to filter the units.  

## Understand the business needs
Dispatchers, Operators and Management are using on a daily the platform, they needs to collaborate with many drivers, clients every day, so it is a bit stressful to use a platform not suited for their special needs.  

# Task  
Organizing the fleet management system  
Track the performance of the fleet  

# Action  
I took the time to ask them about the challenges they faces, what they want to have in the application. this step is very important before designing the solution.  
Then I researched the best features to add, the techniques to acheive this solution.  
I used R Shiny to develop a web based application, connecting with the APIs of Canbus, to customize the entire experience.  
* Prepared data.  
* Recognized valuable datasets.  
* Understand the business workflow.  
* Designed the sytem to be adapted to the users, made it easy to use, and interactive.  
* Automated process: Data collection, manipulation and visualization.  
* Deployed the solution to be web-based.

# Result  
The application is web-based, allowing authenticated users to visually control the moving assets of the company.  
Reduce significantly the time of requests.  
Collaborate effectively with the team, increase the reliability of communication.  
Alerts are visualized on the map, no need for lists.  
  
<!--- ![fleetmap](/images/fleetmap_hamzaimloul.png)  --->
<img src="/images/fleetmap_hamzaimloul.png" width="500px"/>
  
Below, you can experiment with the map layers! 
the client preferred to make content in French. I will explain the meaning of items.
  
{% include base_path %}
<object data="/files/map.html" type="text/html" width="500px" height="300px">
<embed src="/files/map.html" type="text/html">
<p>This browser does not support PDFs. Please download the PDF to view it: <a href="/files/map.html">Download HTML</a>.</p>
</embed>
</object>  
  
**Four Layers**:   
  
| Layer        | Description                                                                                                              |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ |
| `Etat`       | Show items by type of activity                                                                                           |
| `Flotte`     | Show clusters of vehicles, it is useful when the dispatcher needs to know how many vehicles in a geographical zone.      |
| `Inactive`   | Show the inactive units, easy way to expose anomalies.                                                                   |
| `poi`        | the sites of interest.                                                                                                   |
  
