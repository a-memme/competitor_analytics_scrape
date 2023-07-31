# Competitor Analysis ETL
*ETL pipeline creation, utilizing selenium web browser for scraping snapchat desktop UI, pushing data into a g-sheet database and loading into reporting*

# Purpose 
The current project leverages web scraping to extract performance data of "competitor" channels on a social media platform through mimicking an "anonymous user" of the app (via Desktop). As API availability is sparse for this particular social media platform, the ability to effectively scrape information on competitors that are essentially receiving significant distribution from the platform is valuable when conducting competitive/market research in the industry (online media). This data is then cleaned and transformed to prepare for interpretability, and loaded into a Google Sheets file to serve as a database which can be accessed by any BI tool such as Looker, Tableau, etc for reporting. 

## Extract 
The following code utilizes selenium web driver in combination with a headless firefox browser to scrape taxonomical and performance data (i.e subscriber base data) from channels that are being distributed in the "Suggested For You" sidebar. See [Copy_Snapchat_Scrape.ipynb](https://github.com/a-memme/competitor_analytics_scrape/blob/main/Copy_Snapchat_Scrape.ipynb) for more details.

- The functions pictured below are used first to configure the selenium web driver, disabling caching, memory, cookies, etc in attempt to mimic an "anonymous" user on platform. In addition, the proxy_identificaion() function navigates to a website where the proxy ip being used by the driver can be identified for data collecting purposes, demonstrated further in the code base. See below:

![image](https://github.com/a-memme/competitor_analytics_scrape/assets/79600550/a2728075-1a02-47f0-b93c-1217e845a30c)

- The sfy_scrape() function serves only as a piece of the entire puzzle, but is isolated simply so that its purpose can be demonstrated in this analysis. Essentially, it gathers information on all relevant channels appearing through scrolling a (somewhat arbitrary) number of pixels down  the page in a for loop  to load new suggested channels. The channels are stored in a list to be accessed through the full_scrape() function which then loops through all the collected channels in the previous step, gathering all relevant information of interest:

```
  #Metrics Scrape
  creator_list = sfy_scrape()
  #Visiting each profile in the creator list and scraping the desired metrics
  for creator in creator_list:
    browser.get(creator)

    #Get metrics of interest
    webpage = browser.title
    page_list.append(webpage)

    distro_type = 'SFY'
    type_list.append(distro_type)

    content_source = 'Creator Show'
    source_list.append(content_source)

    #Append ip country and city from above steps
    ip_country.append(country)
    ip_city.append(city)

    #Get channel name
    try:
      channel = browser.find_element(By.CLASS_NAME, "PublicProfileDetailsCard_inlineDiv__V12Dg").text
      channel_list.append(channel)
    except NoSuchElementException:
      channel=np.nan
      subs=np.nan
      description = np.nan
      num_eps = np.nan
      landing_episode = np.nan
      landing_info = np.nan
      landing_thumbnail = np.nan

      channel_list.append(channel)
      subs_list.append(subs)
      descrp_list.append(description)
      num_eps_list.append(num_eps)
      landing_ep_list.append(landing_episode)
      landing_info_list.append(landing_info)
      landing_thumb_list.append(landing_thumbnail)
      final_links.append(np.nan)
      time_list.append(np.nan)

      continue

    #Get subscriber base
    try:
      subs = browser.find_element(By.CLASS_NAME, "PublicProfileDetailsCard_desktopSubscriberTextOnMedia__l0rjj").text
    except NoSuchElementException:
      subs=np.nan
    subs_list.append(subs)

    #Get channel description
    description = browser.find_element(By.CLASS_NAME, "PublicProfileCard_desktopTitle__9ik6D").text
    descrp_list.append(description)

    #Get the most recent episodes location, and respective metrics
    #Scroll to bottom of page
    SCROLL_PAUSE_TIME = 5
    # Get scroll height
    last_height = browser.execute_script("return document.body.scrollHeight")
    while True:
        # Scroll down to bottom
        browser.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        # Wait to load page
        time.sleep(SCROLL_PAUSE_TIME)
        # Calculate new scroll height and compare with last scroll height
        new_height = browser.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break
        last_height = new_height

    episodes = browser.find_elements(By.CLASS_NAME, 'StoryListTile_title__uu0Lo')
    num_eps = len(episodes)
    num_eps_list.append(num_eps)

    if num_eps > 0:
      landing_episode = episodes[(num_eps-1)].text

      all_episodes_info = browser.find_elements(By.CLASS_NAME, 'StoryListTile_storyInfo__XnOTC')
      landing_info = all_episodes_info[(num_eps-1)].text

      multiple = browser.find_elements(By.CSS_SELECTOR, ".StoryListTile_thumbnail__NYD_G [src]")
      landing_thumbnail = multiple[(num_eps-1)].get_attribute('src')

    else:
      landing_episode = np.nan
      landing_info = np.nan
      landing_thumbnail = np.nan

    #Append all recent episode metrics to lists
    landing_ep_list.append(landing_episode)
    landing_info_list.append(landing_info)
    landing_thumb_list.append(landing_thumbnail)


    #Append link to final link list
    final_links.append(creator)

    #Timestamp
    utc = pd.Timestamp.today().floor('MIN').to_numpy()
    timestamp = utc - np.timedelta64(4, 'h')
    time_list.append(timestamp)

    browser.delete_all_cookies()
    time.sleep(10)
```

## Transform
Adjustments to data types and regex formatting are applied to some of the data fields in order to prepare the data for loading and interpretation. The final table looks something like what's pictured below:

![image](https://github.com/a-memme/competitor_analytics_scrape/assets/79600550/c6733abe-e2d5-482a-8531-5095327fc123)

## Load 
Finally, using google colab authentication and gspread, the dataframe is pushed into a pre-existing G-sheet to serve as a database where information can be accessed by a BI Tool such as Looker for greater visibility or intepretation. If run on the schedule, differential metrics (such as subscriber growth per day) can be calculated to monitor the performance of competior channels of interest.

![image](https://github.com/a-memme/competitor_analytics_scrape/assets/79600550/f3c24f05-8348-4d86-b013-5b7424052e34)

