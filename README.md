# Paper 1

**Automatically Detecting Vulnerable Websites Before They Turn Malicious**<br>
K. Soska, Nicolas Christin<br>
2014, USENIX Security Symposium<br>

### Paper Summary

- Introduced a novel classification system which predicts whether a currently benign website will become compromised within a year. (using textual and structural analysis of a web page)
- The system is able to detect new attack trends relatively quickly. (the set of features it relies on is automatically extracted from the data it acquires)
- The implemented classifier was able to achieve a 66% True Positive rate, a 17% False Positive rate.

### Methodology

<img src="Blueprints/Automatically Detecting Vulnerable Websites Before They Turn Malicious.png">

### Features 

#### AWIS Features (Static Features)

| Feature | Description | Discretization fn | Values |
|:---|:---|:---|:---|
|AWIS Site Rank |Global and regional popularity rankings |[log(SiteRank)] |[0…8] |
|Links to the site |The number of other sites which link in |[log(LinksIn)] |[0…7] |
|Load percentile |The average time it takes users to load the page |[LoadPercentile/10] |[0…10] |
|Adult site? |Whether it is an adult site |(Boolean) |{0,1} |
|Reach per million |The faction of all users that are exposed to the site |[log(reachPerMillion)] |[0…5] |

#### Content-based Features (Dynamic, Binary Features)

Example:
- meta{’content’: ’WordPress 3.2.1’, ’name’: ’generator’}
- meta{’content’: ’WordPress 3.3.1’, ’name’: ’generator’}
- span{’class’: [’breadcrumbs’, ’pathway’]}
- div{’id’: ’content disclaimer’}
- If comments are open, but there are no comments

### Related Measurement Project [6]

### Content Extraction Algorithm [10]

- User-generated content is all data in a webpage, visible and invisible, which is not part of the template or CMS
- Each web page in a website is broken down into a DOM tree and joined to form a style tree. 
- A DOM tree captures the tags, attributes present in a page (and their relationships).
- Each node in a style tree represents an HTML tag from one/many web pages within the site and has a property called composite importance which indicates how important the node is.
  - CompImp closer to 0 - node appears often in the site 
  - CompImp closer to 1 - node is unique (or has children which are unique))
- To remove user generated content select nodes with CompImp > 0.99 <br>

### Statistic-based Extraction

- The impact of a feature is not known at the time of feature selection
- So, to determine an optimal set of features, statistic s(t) is computed for each tag t in the dictionary.
- The top-200 ranked entries in the dictionary are selected according to the statistic
- ACC2 is used as statistic s(x) which is the balanced accuracy for tag x, defined as:

<img src="https://render.githubusercontent.com/render/math?math=[s(x) = \left|\frac{|\{x : x \in w, w \in \mu\} |} {|\mu|} - \frac{|\{x : x \in w, w \in \beta\} |}{|\beta|}\right|" width="350" height="70">

- Each time the decision tree classifiers are trained, the top features are recomputed to reflect changes as a result of recent examples.
- Calculation of the ACC2 statistic for a feature at a particular time is parameterized by the window size & a weighting scheme.
  - **Windowing**: the dictionary of tags only contains entries from the last K sites. (For a new tag to be considered a top tag, it needs to be observed a large number of times. So there can be a significant delay between when a tag becomes useful for classification and when it will be selected as a top feature)
  - **Weighting scheme**: Features are weighted differently depending on when they are observed. Gives higher weight to more recent examples.

### Limitations 

- The system assumes the factors responsible for whether or not a site will become compromised can be summarized by its content and traffic statistics.
- Adversaries will not systematically attack all sites containing a particular vulnerability. The system will classify sites with similar content to those which were compromised in the campaign as becoming malicious in the future. However, the attackers may have chosen to ignore them.
- In the dynamic feature extraction system, tags often rise to the top of the list because they are part of some page template which has come up frequently. There may be multiple tags associated with a particular page template which all rise at the same time, and so a few of the top tags are redundant since they identify the same thing.
- It will take longer to train the classifiers and run the system for system configurations which use past features in addition to the current top features. (The size of the feature set is monotonically increasing)

<br><br>

# Paper 2

**Cloudy with a Chance of Breach: Forecasting Cyber Security Incidents**<br>
Y. Liu, Armin Sarabi, +4 authors M. Liu<br>
2015, USENIX Security Symposium<br>

### Paper Summary

- Introduced a classification system to proactively forecast an organization’s cyber security breaches based on externally observable properties of an organization’s network.
- Incident types include (Web app, Hijacking, SQLi, DDos, Defacement, Crimeware etc.)
- Implemented a Random Forest (RF) classifier able to achieve a 90% True Positive rate, a 10% False Positive rate, and 90% accuracy.
- Tested the prediction outcome for the top 5 data breaches in 2014 and correctly labeled 4 incidents.

### Methodology

<img src="Blueprints/Cloudy with a Chance of Breach Forecasting Cyber Security Incidents.png">

### Features 

#### Mismanagement Symptoms

|Feature |Description |Novel |
|:---|:---|:---|
|Open Recursive Resolvers |Is the DNS resolver misconfigured? |[11] |
|DNS Source Port Randomization |Does the DNS server implement source port randomization? |[11] |
|BGP Misconfiguration |BGP configuration errors or reconfiguration events |[11] |
|Untrusted HTTPS Certificates |Is the HTTP certificate signed by a browser-trusted certificate authority? |[11] |
|Open SMTP Mail Relays |Does the email server perform filtering on the message source or destination to only allow users in their own domain to send email messages? |[11] |

#### Malicious Activity

##### Time Series
No. of unique IPs (in a unit) blacklisted per day for a 60 day period for each malicious activity (spam, phish, scan)

##### Secondary Features
Statistics summarizing dynamic behavioral patterns extracted from the time series

- Each time series is value-quantized into three regions relative to its time average: “good”, “normal” and “bad”.
  - Time Average - The average magnitude (no. of IPs listed) of the time series over the observation period 
  - Normal Region - Range of magnitude values relatively close to the time-average 
  - Bad region - Large number of listed IPs relative the average 
  - Good region - Small number of listed IPs relative to the average<br> 
- Four summary statistics are derived for each region, resulting in 12 values for each time series (36 derived features)
  1. Average magnitude (normalized by the total number of IPs in an organization)
  2. Average magnitude (unnormalized)
  3. Average duration that the time series persists in that region upon each entry (in days)
  4. Frequency at which the time series enters that region.<br>
- These statistics are further distinguished as,
  - Recent-60 features (36) - Collected over a period of 60 days (typically leading up to the time of incident) 
  - Recent-14 features (36) - Collected over a period of 14 days (leading up to the time of incident).<br>

### Strengths 
- The trained random forest classifier achieved 90% true positive rate and 10% false positive rate. 
- The prediction results substantially outperform what has been shown in the literature to date [1]
- Prediction on the organizational level can help point to networks facing heightened probability of a broader class of security problems.

### Limitations
- Big corporations tend to register their IP address blocks under multiple owner IDs. Therefore, processing these IDs in the system will result in them being treated as separate organizations.
- Since security incidents are largely under-reported, 
  - All factors affecting an organization’s likelihood of suffering a breach might not be identified because all incidents are not included in the training set
  - Non-victim organizations might be selected from unreported victims
	These factors will affect the accuracy of the classifier 
