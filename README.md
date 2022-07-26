# YouTube Scraping

Hi there! Welcome, I worked my hands on a Web Scraping Project using Python. Below you'll find the entire code which you may as well use for your portfolio projects.

In the Resources folder above, you'll find:
* Jupyter Notebook with Python Code (.ipynb)
* The Power BI DashBoard file (.pbix)

You may as well [watch the video]() (coming soon) to get a better idea of what's in the bag.

![pexels-pixabay-39284](https://user-images.githubusercontent.com/91784043/180906041-e310202f-6c4a-4d3c-9a31-7cba80f2f697.jpg)


### Importing Libraries

    from googleapiclient.discovery import build
    import pandas as pd
    import seaborn as sns
    import matplotlib.pyplot as plt
    import cufflinks as cf

    %matplotlib inline
    cf.go_offline()
    
### Getting the API Key

    api_key = '' #Enter your own API key, know more by watching the video linked above.

### Channel IDs

    channel_ids = ['UCHnyfMqiRRG1u-2MsSQLbXA', # Veritasium
                   'UCsooa4yRKGN_zEE8iknghZA', # Ted-Ed
                   'UCOajpsI8t3Eg-u-s2j_c-cQ', # Movie Flame
                   'UC-CSyyi47VX1lD9zyeABW3w', # Dhruv Rathee 
                   ''  # VSauce
                  ]

    youtube = build('youtube', 'v3', developerKey=api_key)
    
### Channel Statistics
 
     def get_channel_stats(youtube, channel_ids):
        all_data = []
        request = youtube.channels().list(
                    part='snippet,contentDetails,statistics',
                    id=','.join(channel_ids))
        response = request.execute() 

        for i in range(len(response['items'])):
            data = dict(Channel_name = response['items'][i]['snippet']['title'],
                        Subscribers = response['items'][i]['statistics']['subscriberCount'],
                        Views = response['items'][i]['statistics']['viewCount'],
                        Total_videos = response['items'][i]['statistics']['videoCount'],
                        playlist_id = response['items'][i]['contentDetails']['relatedPlaylists']['uploads'])
            all_data.append(data)

        return all_data
        
        
        
        channel_statistics = get_channel_stats(youtube, channel_ids)
        
        
        
        channel_data = pd.DataFrame(channel_statistics)
        
        
        channel_data
        
### Convert into a .csv file

    channel_data.to_csv('channel_stats.csv', index = False)
    
### Data Cleaning

    channel_data['Subscribers'] = pd.to_numeric(channel_data['Subscribers'])
    channel_data['Views'] = pd.to_numeric(channel_data['Views'])
    channel_data['Total_videos'] = pd.to_numeric(channel_data['Total_videos'])
    channel_data.dtypes
    
### Exploratory Data Analysis
    
    # I invite you to do what you would like to do with this data, I have shared what I did!
    
    # Subscribers per Channel
    channel_data.sort_values(by=['Subscribers']).iplot(kind='bar', x='Channel_name', y='Subscribers', xTitle='Channel Name',
                                                       yTitle='Subscribers')
                                                   
    
    # Views per Channel
    channel_data.iplot(kind='bar', x='Channel_name', y='Views', xTitle='Channel Name',
                       yTitle='Views')
                       
    # Total Video per Channel
    channel_data.iplot(kind='bar', x='Channel_name', y='Total_videos', xTitle='Channel Name',
                       yTitle='Total Videos')


### Function to get Video IDs

    # Get the playlist ID
    playlist_id = channel_data.loc[channel_data['Channel_name']=='Veritasium', 'playlist_id'].iloc[0]
    
    # The main function
    def get_video_ids(youtube, playlist_id):
    
    request = youtube.playlistItems().list(
                part='contentDetails',
                playlistId = playlist_id,
                maxResults = 50)
    response = request.execute()
    
    video_ids = []
    
    for i in range(len(response['items'])):
        video_ids.append(response['items'][i]['contentDetails']['videoId'])
        
    next_page_token = response.get('nextPageToken')
    more_pages = True
    
    while more_pages:
        if next_page_token is None:
            more_pages = False
        else:
            request = youtube.playlistItems().list(
                        part='contentDetails',
                        playlistId = playlist_id,
                        maxResults = 50,
                        pageToken = next_page_token)
            response = request.execute()
    
            for i in range(len(response['items'])):
                video_ids.append(response['items'][i]['contentDetails']['videoId'])
            
            next_page_token = response.get('nextPageToken')
        
    return video_ids
    
    # Store the values in a varaible 
    video_ids = get_video_ids(youtube, playlist_id)
    
### Function to get video details

    # Main Function
    def get_video_details(youtube, video_ids):
    all_video_stats = []
    
    for i in range(0, len(video_ids), 50):
        request = youtube.videos().list(
                    part='snippet,statistics',
                    id=','.join(video_ids[i:i+50]))
        response = request.execute()
        
        for video in response['items']:
            video_stats = dict(Title = video['snippet']['title'],
                               Published_date = video['snippet']['publishedAt'],
                               Views = video['statistics']['viewCount'],
                               Likes = video['statistics']['likeCount'],
                               
                               Comments = video['statistics']['commentCount']
                               )
            all_video_stats.append(video_stats)
    
    return all_video_stats
    
    # Store the Values in a variable
    video_details = get_video_details(youtube, video_ids)
    
    # Creating a dataframe off of it
    video_data = pd.DataFrame(video_details)
    
    # Data Pre-Processing
    video_data['Published_date'] = pd.to_datetime(video_data['Published_date']).dt.date
    video_data['Views'] = pd.to_numeric(video_data['Views'])
    video_data['Likes'] = pd.to_numeric(video_data['Likes'])

    video_data['Views'] = pd.to_numeric(video_data['Views'])
    video_data
    
    # Let's fetch the Top 10 Videos
    top10_videos = video_data.sort_values(by='Views', ascending=False).head(10)
    sns.barplot(x='Views', y='Title', data=top10_videos)
    
![image](https://user-images.githubusercontent.com/91784043/180907361-cbd6913c-e2ac-4b5e-be7f-36ab8bd93fe7.png)

<br/><br/>

# Dashboard

![image](https://user-images.githubusercontent.com/91784043/180908491-93205f5f-265b-4780-be6a-829623e7171a.png)

