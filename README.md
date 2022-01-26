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

#### Related Measurement Project [6]

#### Content Extraction Algorithm [10]

- User-generated content is all data in a webpage, visible and invisible, which is not part of the template or CMS
- Each web page in a website is broken down into a DOM tree and joined to form a style tree. 
- A DOM tree captures the tags, attributes present in a page (and their relationships).
- Each node in a style tree represents an HTML tag from one/many web pages within the site and has a property called composite importance which indicates how important the node is.
  - CompImp closer to 0 - node appears often in the site 
  - CompImp closer to 1 - node is unique (or has children which are unique))
- To remove user generated content select nodes with CompImp > 0.99 <br>

#### Statistic-based Extraction

- The impact of a feature is not known at the time of feature selection
- So, to determine an optimal set of features, statistic s(t) is computed for each tag t in the dictionary.
- The top-200 ranked entries in the dictionary are selected according to the statistic
- ACC2 is used as statistic s(x) which is the balanced accuracy for tag x, defined as:

<img src="https://render.githubusercontent.com/render/math?math=[s(x) = \left|\frac{|\{x : x \in w, w \in \mu\} |} {|\mu|} - \frac{|\{x : x \in w, w \in \beta\} |}{|\beta|}\right|" width="350" height="70">

Where<br>
&beta; :  the set of benign websites<br>
&mu; : the set of malicious websites<br>
x ∊ w : the tag x is present in the tag dictionary associated with website w<br> 

- The statistic computes the absolute value of the difference between the tag frequency in malicious pages and the tag frequency in benign pages.
- Each time the decision tree classifiers are trained, the top features are recomputed to reflect changes as a result of recent examples.
- Calculation of the ACC2 statistic for a feature at a particular time is parameterized by the window size & a weighting scheme.
  - **Windowing**: the dictionary of tags only contains entries from the last K sites. (For a new tag to be considered a top tag, it needs to be observed a large number of times. So there can be a significant delay between when a tag becomes useful for classification and when it will be selected as a top feature)
  - **Weighting scheme**: Features are weighted differently depending on when they are observed. Gives higher weight to more recent examples.


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
- meta{'content': 'WordPress 3.2.1', 'name': 'generator'}
- meta{'content': 'WordPress 3.3.1', 'name': 'generator'}
- span{'class': ['breadcrumbs’,’pathway']}
- div{'id': 'content disclaimer'}
- If comments are open, but there are no comments


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

<br><br>

# Paper 3

**Identifying Risk Factors for Web Server Compromise**<br>
Marie Vasek, T. Moore<br>
2014, Financial Cryptography<br>

### Paper Summary

- Identified attributes of web servers (notably content management systems) that affect the susceptibility to compromise through a case control study.
- Explored the effect of Content Management Systems, server software, and webserver hygiene indicators on compromise rate.
- Found that web servers running WordPress and Joomla are more likely to be hacked than those not running any CMS. 
- Found that servers running Apache and Nginx are more likely to be hacked than those running Microsoft IIS.
- Observed that a CMS’s market share is positively correlated with website compromise.
- Discovered that servers running outdated versions of WordPress are less likely to be hacked than those running more recent versions.
- Observed that running on a shared host is a positive risk factor for being hacked to serve phishing pages. However, it is a negative risk factor for being hacked for search-redirection attacks. 

### Methodology

<img src="Blueprints/Identifying Risk Factors for Web Server Compromise.png">

#### Case-Control Study

Sample a population of web servers and compare them to other populations of web servers that have been compromised to identify important risk factors by comparing the relative incidence of different characteristics in the case and control populations

#### Odds ratios 

Compare the odds of a subject in the case population exhibiting a risk factor to the odds of a subject in the control population exhibiting a risk factor
- 1 : No difference in proportions of the risk factor among the case and control groups
- \> 1 : Those in the case group are more likely to exhibit the risk factor (positive risk factors)
- < 1 : Those in the case group are less likely to exhibit the risk factor (negative risk factors)

### Features 

#### Explanatory Variables for Logistic Regression

|Category |Feature |Description |Novel |
|:---|:---|:---|:---|
|CMS Market Share |#Servers |Market share of CMS (as of Jan 1, 2013) multiplied by population of registered .com (106.2M) domains and estimated server response rate (85%) |[12][13] |
|Webserver Hygiene |HTTPONLY |Is the HTTPONLY cookie set in the header? |[14] |
| |Server Vsn |Does the server header provide any potentially valid version information? |✔
| |Shared Hosting |Is the domain part of a shared host? (Check if 10 domains in the combined dataset resolve to the same IP address) |[15] |
|Server Attributes |Country |Country of origin |✔ |
| |Server Type |The type of server software (Apache, Microsoft IIS, Nginx, Google, and Yahoo) |✔ |

### Limitations

- There is a delay between the time of reported compromise and the identification of risk factors. It is possible that some of the web servers may have changed their configurations before all indicators could be gathered.
- The study considers only .com domains  
- The findings of the study are not complemented by other forms of experimentation that directly isolate explanatory factors.

<br><br>

# Paper 4

**Evil Searching: Compromise and Re-compromise of Internet Hosts for Phishing**<br>
T. Moore, R. Clayton<br>
2009, Financial Cryptography<br>

### Paper Summary

- Studied and demonstrated the use of search terms and search engines to seek out vulnerable web servers.
- Established that at least 18% of website compromises are triggered by such searches.
- Found that 19% of phishing websites are re-compromised within six months.  
- Discovered that the rate of re-compromise is much higher (48%) if the phishing websites have been identified through web search 
- Observed that phishing websites placed onto a public blacklist are re-compromised no more frequently than websites only known within closed communities.
- Discussed strategies for mitigating 'evil search' attacks

### Methodology

<img src="Blueprints/Evil Searching Compromise and Re-compromise of Internet Hosts for Phishing.png">

<br><br>

# Paper 5

**Measuring and Analyzing Search-Redirection Attacks in the Illicit Online Prescription Drug Trade**<br>
Nektarios Leontiadis, Nicolas Christin<br>
2011, USENIX Security Symposium<br>

### Paper Summary

- Investigated search-redirection attacks where miscreants compromise high-ranking websites and dynamically redirect traffic to online pharmacies over a 9 month period
- Found that one third of all search results are one of over 7 000 infected hosts that redirect to a few hundred pharmacy websites    
- Established that Infections persist longest on websites with high PageRank and from .edu domains    
- Found that 96% of infected domains are connected through traffic redirection chain   
- Estimated a conversion rate of 0.3% - 3% for drug searches turning into sales

### Methodology

<img src="Blueprints/Measuring and Analyzing Search-Redirection Attacks in the Illicit Online Prescription Drug Trade.png">

#### Impact measure 

- Adds up the number of times a domain appears, while compensating for the relative ranking of the search results. 
- When a domain appears as the top result it is much more likely to be utilized than if it appeared on page four of the results. The heuristic normalizes the top result to 1, and discounts the weighting by half as the position drops by 10. (results appearing on page one regarded as twice as valuable as those on page two and so on)

<img src="https://render.githubusercontent.com/render/math?math=\Gamma(domain) = \sum_{q \in queries} \sum_{d \in days} u_{qd} * 0.5^{\frac{r_{qd}-1}{10}}" width="350" height="70">

Where<br>
u<sub>qd</sub> : 1 if domain in result of query q on day d & actively redirects to pharmacy<br>
u<sub>qd</sub> : 0 otherwise<br>
r<sub>qd</sub> : domain's position is search results<br> 

#### Survival function 

The survival function S(t) measures the probability that the infection’s lifetime is greater than time t. (standard Kaplan-Meier estimator)

<br><br>

# Paper 6

**A Nearly Four-Year Longitudinal Study of Search-Engine Poisoning**<br>
Nektarios Leontiadis, T. Moore, Nicolas Christin<br> 
2014, CCS<br>

### Paper Summary

- Investigated the evolution of search-engine poisoning using data on over 5 million search results collected over nearly 4 years.
- Studied the efforts of hosts to remedy search-redirection attacks.
- Found that the median time to clean up source infections has fallen from around 30 to 15 days from 2010 to 2013, but the number of distinct infections has considerably increased.
- Found that a small number of traffic brokers funnel traffic from compromised websites to destinations (illicit pharmacies) and they are concentrated over a few autonomous systems.
- Observed that miscreants have readily adopted to the efforts of search engines and browsers to demote low quality content and protect the privacy of search queries 

### Methodology

<img src="Blueprints/A Nearly Four-Year Longitudinal Study of Search-Engine Poisoning.png">

### Limitations: 

- The study only looked at Google search results 
- The study considered search results based on their presence or not in the results. However, their position in the results is more important

<br><br>

# Paper 7

**Catching Predators at Watering Holes: Finding and Understanding Strategically Compromised Websites**<br>
Sumayah Alrwais, Kan Yuan, Eihal Alowaisheq, Xiaojing Liao, Alina Oprea, XiaoFeng Wang and Zhou Li<br>
2016, ACSAC<br>

### Paper Summary

- Discovered and analyzed new instances of an attack type called "watering hole"
- In a watering hole attack the adversary carefully chooses the target frequently visited by an organization or a group of individuals to compromise, for the purpose of gaining a step closer to the organization or collecting information from the group.
- Discovered and confirmed 17 watering holes and 6 attack campaigns never reported before.
- Gained deeper understanding of these attacks, such as repeated compromises of political websites, their long lifetimes, unique evasion strategy and new exploit techniques.
- Discovered a recent JSONP attack on an NGO website (The attack was forced to stop as a result of their reporting)
- Introduced a methodology to find watering hole instances by analyzing the headers of HTTP requests of target websites to identify suspicious changes. This approach is based on the idea that visiting a website multiple times in a short period of time rarely results in different HTTP requests

### Methodology

<img src="Blueprints/Catching Predators at Watering Holes Finding and Understanding Strategically Compromised Websites.png">

#### Model for Change Rate

- Traffic history includes features extracted from HTTP requests like sub domains, URLs, content types, URL patterns etc. (Assume m features f1…….fm)
- For each additional visit to the site, the HTTP requests are inspected and feature values are compared.
- Feature Fi’s change rate (Ni) is the number of new values observed for Fi (values not observed in the profile) 
- The visit’s overall change score (for the nth visit) is defined as:

<img src="https://render.githubusercontent.com/render/math?math=R_n = \sum_{j=1}^1 N_j" width="150" height="50">

- Probabilistic model (Pn) is constructed for change rates R1….Rn of n continuous visits stored in the profile.
- Pn represents the expected distribution for visit’s change rates i.e. the probabilities Pr[Rn <= k] for all integer values of k
- Confidence intervals are built at different confidence levels 

### Features: 

- A profile is used to keep track of a target’s HTTP traffic history and its change rates. 
- The history is a collection of features extracted from HTTP headers and their values including,
	- FQDN (All domain names that appear in clean visits)
	- Sub-domains
	- Content types (All distinct content types the web site serves)
	- URLs
	- URL patterns per subdomain 
	- File names per subdomain

<br><br>

# Paper 8

**deSEO: Combating Search-Result Poisoning**<br>
J. John, Y. Xie, A. Krishnamurthy, and M. Abadi<br>
2011, USENIX Security Symposium<br>

### Paper Summary

- Performed an in-depth study of SEO attacks that spread malware by poisoning search results for popular queries 
- Examined a live large scale search poisoning attack and studied the methods used by attackers
- Developed a system called deSEO that automatically detects search result poisoning attacks without crawling the content of web pages (Studying just the structure of URLs)
- deSEO detects groups of malicious URLs, where each group correspond to an SEO campaign

### Methodology

<img src="Blueprints/deSEO Combating Search-Result Poisoning.png">

When aggregating URL features by domain name, sub-domains were considered separately because it is possible for a subdomain to get compromised rather than the entire domain.

### Features: 
 
Lexical features of URLs:
1. **String features**: 
	- Separator between keywords
	- Argument name
	- Filename
	- Subdirectory name before the keywords
3. **Numerical features**: 
	- Number of arguments in the URL
	- Length of arguments
	- Length of filename
	- Length of keywords.
5. **Bag of words**: Keywords.







