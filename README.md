# Competitor Analysis ETL
*ETL pipeline creation, utilizing selenium web browser for scraping snapchat desktop UI, pushing data into a g-sheet database and loading into reporting*

# Purpose 
The current project leverages web scraping to extract perforance data of "competitor" channels on a social media platform that are being distributed to what may be considered an anonymous user of the app (via Desktop). As API availability is sparse for this particular social media platform, the ability to effectively scrape information on competitors that are essentially receiving significant distribution from the platform is valuable when conducting competitive/market research in the industry (online media). This data is then cleaned and transformed to prepare for interpretability, and loaded into a Google Sheets file to serve as a database which can be accessed by/tied to any BI tool such as Looker, Tableau, etc for reporting. 

## Extract 
The following code utilizes selenium web driver in combination with a headless firefox browser to scrape taxonomical and performance data (i.e subscriber base data) from channels that are being distributed in the "Suggest For You" sidebar. See [Copy_Snapchat_Scrape.ipynb](https://github.com/a-memme/competitor_analytics_scrape/blob/main/Copy_Snapchat_Scrape.ipynb) for more details.

- The functions pictured below are used first to configure the selenium web driver, disabling caching, memory, cookies, etc in attempt to mimic an "anonymous" user on platform. In addition, the proxy_identificaion() function navigates to a website where the proxy ip being used by the driver can be identified for data collecting purposes, demonstrated further in the code base. See below:

![image](https://github.com/a-memme/competitor_analytics_scrape/assets/79600550/a2728075-1a02-47f0-b93c-1217e845a30c)

- The sfy_scrape() function serves only as a piece of the entire puzzle, but is isolated simply so that its purpose can be demonstrated in this analysis. Essentially, it gathers information on all relevant channels appearing through scrolling a (somewhat arbitrary) number of pixels down  the page in a for loop  to load new suggested channels. The channels are stored in a list to be accessed through the full_scrape() function which then loops through all channels gathering all information of interest:

#Update
! apt-get update
#Temporarily install firefox
! apt install firefox  xvfb > /dev/null
#Selenium
! pip3 install  pyvirtualdisplay selenium webdriver_manager > /dev/null
