##  YouTube Data Engineering Project
This project focuses on building a data engineering pipeline to retrieve data from the YouTube API, load it into a CSV file, then into a PostgreSQL database, and finally perform data analysis. The goal is to extract insights from the data and document findings in a report.

#### Project Workflow
1. Access YouTube API: We will start by obtaining data from the YouTube API. Please ensure you have a valid API key to access and retrieve the necessary data.

2. Load Data into CSV: After retrieving the data, we will format and save it in a CSV file for easy access and backup.
   
3. Load Data into PostgreSQL: Next, we will import the CSV data into a PostgreSQL database for efficient querying and storage.

4. Analyze Data: Using a data analysis library, we’ll explore and analyze the data in the DataFrame to identify trends, patterns, and any notable insights.

5. Generate Report: The findings from the analysis will be documented in a report that highlights key observations and insights.

   
#### Tools and Environment
Google Colab: We’ll use Google Colab to execute the project for convenience and simplicity, especially for accessing Jupyter notebooks and data libraries.

PostgreSQL: Our database of choice for storing the YouTube data.

Python Libraries: Pandas, SQLAlchemy, and other essential libraries for data handling and analysis.

Getting Started
To begin, make sure to have access to the YouTube API and set up Google Colab for running the project. Follow the detailed steps provided in each notebook to walk through the data extraction, transformation, loading, and analysis stages.

```
pip install google-api-python-client
from googleapiclient.discovery import build
import seaborn as sns
import pandas as pd
```
These will be the libraries we will be using for our project

```
api_key ='' # get your api key from here
channel_ids =[''] # put your channel ids of he youube channels you want to analyze

youtube =build('youtube','v3',developerKey =api_key)
```
api_key = '':

This variable stores your YouTube API key, which is necessary for authenticating requests to the YouTube Data API. You can obtain the API key from your Google Cloud Console.
channel_ids = ['']:

This list contains the unique channel IDs for the YouTube channels you want to analyze. You can add multiple channel IDs within the list to retrieve data for various channels.
youtube = build('youtube', 'v3', developerKey=api_key):

This line initializes the YouTube Data API client using the build function from the Google API client library.
The function parameters specify:


'youtube': The service name for YouTube.
'v3': The version of the YouTube API.


developerKey=api_key: The API key, allowing access to the API’s resources.
The youtube object created here can be used to make requests to retrieve data from YouTube for the specified channels.

```
def get_channel_stats(youtube, channel_ids):
    all_data = []

    # API request to get channel details
    request = youtube.channels().list(
        part='snippet,contentDetails,statistics',
        id=','.join(channel_ids)
    )
    response = request.execute()

    # Loop through the response to extract relevant details
    for item in response['items']:
        data = {
            'channel_name': item['snippet']['title'],
            'Subscribers': item['statistics']['subscriberCount'],
            'Views': item['statistics']['viewCount'],
            'Total_videos': item['statistics']['videoCount'],
            'playlist_id': item['contentDetails']['relatedPlaylists']['uploads']
        }
        all_data.append(data)

    return all_data
```
Here’s an explanation of the `get_channel_stats` function:

1. **Function Definition**:
   - **`def get_channel_stats(youtube, channel_ids):`**: This function takes two arguments: `youtube` (the YouTube API client object) and `channel_ids` (a list of YouTube channel IDs to analyze). It will retrieve specific statistics and details about each channel.

2. **Initialize Data Storage**:
   - **`all_data = []`**: Creates an empty list to store the data for each channel in a structured format (dictionaries in this case).

3. **API Request to Get Channel Details**:
   - **`request = youtube.channels().list(...)`**:
     - This line makes an API request to YouTube to get details about the channels specified in `channel_ids`.
     - The `part` parameter includes `'snippet'`, `'contentDetails'`, and `'statistics'`, which specify what information to retrieve about each channel:
       - `snippet`: Contains general information about the channel (e.g., title).
       - `contentDetails`: Includes data like playlists related to the channel.
       - `statistics`: Holds statistical data about the channel (e.g., subscriber count, view count).
     - **`id=','.join(channel_ids)`**: Joins all channel IDs into a comma-separated string to include them in the request.

   - **`response = request.execute()`**: Executes the API request and stores the JSON response in `response`.

4. **Loop Through Response and Extract Details**:
   - **`for item in response['items']:`**: Iterates over each item (channel) in the response.
   - **Data Extraction**:
     - For each channel, it extracts:
       - **`channel_name`**: Channel title from `item['snippet']['title']`.
       - **`Subscribers`**: Subscriber count from `item['statistics']['subscriberCount']`.
       - **`Views`**: Total view count from `item['statistics']['viewCount']`.
       - **`Total_videos`**: Total video count from `item['statistics']['videoCount']`.
       - **`playlist_id`**: ID of the channel’s upload playlist from `item['contentDetails']['relatedPlaylists']['uploads']`.
   - **Store Extracted Data**:
     - Each channel's data is stored in a dictionary `data`, and then appended to the `all_data` list.

5. **Return Results**:
   - **`return all_data`**: The function returns `all_data`, a list of dictionaries containing stats and details for each channel.

```
def get_video_ids(youtube, playlist_id):
    request = youtube.playlistItems().list(
        part='contentDetails',
        playlistId=playlist_id,
        maxResults=50
    )
    response = request.execute()

    video_ids = []

    for i in range(len(response['items'])):
        video_ids.append(response['items'][i]['contentDetails']['videoId'])

    next_page = response.get('nextPageToken')
    more_pages = True

    while more_pages:
        if next_page is None:
            more_pages = False
        else:
            request = youtube.playlistItems().list(
                part='contentDetails',
                playlistId=playlist_id,
                maxResults=50,
                pageToken=next_page
            )
            response = request.execute()

            for i in range(len(response['items'])):
                video_ids.append(response['items'][i]['contentDetails']['videoId'])

            next_page = response.get('nextPageToken')

    return (video_ids)
```
Here’s an explanation of the `get_video_ids` function:

1. **Function Definition**:
   - **`def get_video_ids(youtube, playlist_id):`**: This function takes two arguments: `youtube` (the YouTube API client object) and `playlist_id` (the ID of the YouTube playlist to retrieve video IDs from). It will collect the video IDs for all videos in the specified playlist.

2. **Initial API Request**:
   - **`request = youtube.playlistItems().list(...)`**:
     - This line creates an API request to retrieve the video IDs from a given playlist.
     - The `part='contentDetails'` parameter specifies that only `contentDetails` (including video ID) should be returned.
     - **`playlistId=playlist_id`**: The playlist ID we’re querying.
     - **`maxResults=50`**: Requests the maximum number of items (50) per API response, which is the limit for YouTube’s API.

   - **`response = request.execute()`**: Executes the API request and stores the JSON response in `response`.

3. **Extract Video IDs**:
   - **`video_ids = []`**: Initializes an empty list to store the video IDs.
   - **`for i in range(len(response['items'])):`**: Iterates over the videos in the response.
   - **`video_ids.append(response['items'][i]['contentDetails']['videoId'])`**: Extracts each video ID from `contentDetails` and appends it to `video_ids`.

4. **Handling Pagination**:
   - **`next_page = response.get('nextPageToken')`**: Checks if there are more pages by retrieving the `nextPageToken` from the response. This token is used to get the next set of results if there are more videos in the playlist.
   - **`more_pages = True`**: Sets up a loop to handle pagination.

   - **`while more_pages:`**: The loop continues until there are no more pages.
     - **`if next_page is None:`**: If `nextPageToken` is `None`, there are no more pages, so it sets `more_pages` to `False` to stop the loop.
     - **Else Condition**:
       - If there is a `nextPageToken`, another API request is made to retrieve the next set of video IDs.
       - **`pageToken=next_page`**: This specifies the page token to get the next set of results.
       - **For Loop**: Extracts the video IDs from each item in the response and appends them to `video_ids`.
       - Updates `next_page` with the next `nextPageToken` from the response.

5. **Return Results**:
   - **`return video_ids`**: The function returns the complete list of `video_ids` collected from the playlist, which may contain multiple pages of results.
  
   ```
   def get_video_details(youtube, video_ids):
    all_video_stats = []

    # Process video IDs in batches of 50 (YouTube API limit)
    for i in range(0, len(video_ids), 50):
        request = youtube.videos().list(
            part='snippet,statistics',
            id=','.join(video_ids[i:i+50])  # Select 50 video IDs at a time
        )
        response = request.execute()

        # Extract video statistics for each video
        for video in response['items']:
            video_stats = {
                'Title': video['snippet']['title'],
                'Published_date': video['snippet']['publishedAt'],

                'Views': video['statistics'].get('viewCount', 0),
                'Likes': video['statistics'].get('likeCount', 0),
                'Dislikes': video['statistics'].get('dislikeCount', 0),
                'Comments': video['statistics'].get('commentCount', 0)
            }
            all_video_stats.append(video_stats)

    return all_video_stats
   ```
   Here’s an explanation of the `get_video_details` function:

1. **Function Definition**:
   - **`def get_video_details(youtube, video_ids):`**: This function takes two arguments: `youtube` (the YouTube API client object) and `video_ids` (a list of video IDs to retrieve details for). The function collects specific details and statistics for each video.

2. **Initialize Data Storage**:
   - **`all_video_stats = []`**: An empty list to store the statistics and details for each video.

3. **Process Video IDs in Batches**:
   - **`for i in range(0, len(video_ids), 50):`**: Loops through `video_ids` in batches of 50 because the YouTube API has a maximum limit of 50 IDs per request.

4. **API Request for Each Batch**:
   - **`request = youtube.videos().list(...)`**:
     - Creates an API request to get video details for the current batch of 50 video IDs.
     - **`part='snippet,statistics'`**: Specifies that both `snippet` (basic details about the video) and `statistics` (like views and likes) should be returned.
     - **`id=','.join(video_ids[i:i+50])`**: Joins up to 50 video IDs into a comma-separated string to include them in the request.

   - **`response = request.execute()`**: Executes the API request and stores the JSON response in `response`.

5. **Extract Video Statistics**:
   - **`for video in response['items']:`**: Iterates over each video in the response.
   - **`video_stats = { ... }`**:
     - Creates a dictionary `video_stats` to store relevant details and statistics for each video:
       - **`Title`**: The video title from `video['snippet']['title']`.
       - **`Published_date`**: The publication date of the video from `video['snippet']['publishedAt']`.
       - **`Views`**: The view count from `video['statistics'].get('viewCount', 0)`. The `get` method is used here with a default value of `0` to handle cases where the view count might be unavailable.
       - **`Likes`**: The like count from `video['statistics'].get('likeCount', 0)`.
       - **`Dislikes`**: The dislike count from `video['statistics'].get('dislikeCount', 0)`.
       - **`Comments`**: The comment count from `video['statistics'].get('commentCount', 0)`.

   - **`all_video_stats.append(video_stats)`**: Adds the `video_stats` dictionary to the `all_video_stats` list.

6. **Return Results**:
   - **`return all_video_stats`**: The function returns `all_video_stats`, a list of dictionaries containing the statistics and details for each video.
  
     These is a summary of how we can use the youube api to get the data for our project.
