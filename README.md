# Youtube-in-Power-BI
*A Personal Dashboard to Track and Visualize Favorite YouTube Channels using YouTube Data API and Power BI*

![Youtube dashboard project pages_page-0001](https://github.com/user-attachments/assets/4319d90a-5033-4e33-b081-4fe7791a3c20)


## Overview
This project connects the YouTube Data API to Power BI to create an interactive dashboard that visualizes data from my favorite YouTube channels.
The dashboard displays:
- The latest videos from selected channels
- Channel-level metrics (subscribers, views, uploads)
- Video-level analytics (titles, likes, views, publish date)
- Custom visuals that allows you to watch the videos in Power BI service.

This is a perfect beginner-to-intermediate API + Power BI project that blends data engineering, visualization, and automation.

## Tech Stack 
| Tool                           | Purpose                                                                   |
| ------------------------------ | ------------------------------------------------------------------------- |
| **Power BI Desktop / Service** | Data connection, modeling, visualization and publishing                           |
**Power Query**                   |   Data cleaning and transformation
| **YouTube Data API v3**        | Fetching channels and videos metadata                                       |
| **Figma**                     | Wireframing and designing the Youtube UI |
| **Google Cloud Console**       | API key management                                                        |

## Project Features
1. Displays a list of my favorite YouTube channels and description
2. Shows channel statistics (subscriber count, total views, total videos)
3. Lists the 50 most recent uploads per channel, and also a link to watch
4. Auto-refresh via Power BI service
5. Clean and interactive layout, resembling the Youtube interface

![Youtube dashboard project pages_page-0002](https://github.com/user-attachments/assets/d8fc7dbf-93d0-4140-bfb8-8fc53419fb65)

## Project Walkthrough
### Getting Started
To begin, I created a new project in Google Cloud Console, and enabled the Youtube API. The API key will be used to authenticate connections later in the project. 

![Screenshot_28-10-2025_11521_console cloud google com](https://github.com/user-attachments/assets/acb361b9-fd3f-4141-9a7b-914fad5d8dca)

The next step is making a list of YouTube channels I want to track. I choose my favorite channels at the time of working on this project, and they are: 
1. Alex The Analyst
2. Mr. Beast
3. Marques Brownlee
4. Lawrence Oyor
5. TY Bello
6. Tayo Aina

You can find the `channel ID` by visiting a Youtube channel, click on 'more' in the description, click on 'Share channel', and then 'copy channel ID'

<img width="940" height="903" alt="image" src="https://github.com/user-attachments/assets/c3b5626a-2b9f-4bad-b0f2-2687eff64c35" />

To get the channel statistics and videos list, I then used my API key and Channel ID to create a connection in Power Query. Here's the code I modified:
**Get Channel Statistics**
```
https://www.googleapis.com/youtube/v3/channels?part=snippet,statistics&id={CHANNEL_ID}&key={YOUR_API_KEY}
```
**Get Videos from a Channel**
```
https://www.googleapis.com/youtube/v3/search?key={YOUR_API_KEY}&channelId={CHANNEL_ID}&part=snippet,id&order=date&maxResults=10
```
After connecting the API to Power Query, and returning my videos list, I then performed some transformations. 

## Data Transformation
*PS: This is for a single channel. You will have to repeat the same steps for as many channels you'll be dealing with.*
*The API key has also been masked (****....****)*

### Transforming the videos list
```
let
    Source = Json.Document(Web.Contents("https://www.googleapis.com/youtube/v3/search?part=snippet&channelId=UC7cs8q-gJRlGwj4A8OmCmXg&maxResults=10000&order=date&type=video&key=****************************************")),
    #"Converted to Table" = Table.FromRecords({Source}),
    #"Expanded pageInfo" = Table.ExpandRecordColumn(#"Converted to Table", "pageInfo", {"totalResults", "resultsPerPage"}, {"pageInfo.totalResults", "pageInfo.resultsPerPage"}),
    #"Expanded items" = Table.ExpandListColumn(#"Expanded pageInfo", "items"),
    #"Expanded items1" = Table.ExpandRecordColumn(#"Expanded items", "items", {"kind", "etag", "id", "snippet"}, {"items.kind", "items.etag", "items.id", "items.snippet"}),
    #"Expanded items.id" = Table.ExpandRecordColumn(#"Expanded items1", "items.id", {"kind", "videoId"}, {"items.id.kind", "items.id.videoId"}),
    #"Expanded items.snippet" = Table.ExpandRecordColumn(#"Expanded items.id", "items.snippet", {"publishedAt", "channelId", "title", "description", "thumbnails", "channelTitle", "liveBroadcastContent", "publishTime"}, {"items.snippet.publishedAt", "items.snippet.channelId", "items.snippet.title", "items.snippet.description", "items.snippet.thumbnails", "items.snippet.channelTitle", "items.snippet.liveBroadcastContent", "items.snippet.publishTime"}),
    #"Expanded items.snippet.thumbnails" = Table.ExpandRecordColumn(#"Expanded items.snippet", "items.snippet.thumbnails", {"default", "medium", "high"}, {"items.snippet.thumbnails.default", "items.snippet.thumbnails.medium", "items.snippet.thumbnails.high"}),
    #"Expanded items.snippet.thumbnails.default" = Table.ExpandRecordColumn(#"Expanded items.snippet.thumbnails", "items.snippet.thumbnails.default", {"url", "width", "height"}, {"items.snippet.thumbnails.default.url", "items.snippet.thumbnails.default.width", "items.snippet.thumbnails.default.height"}),
    #"Expanded items.snippet.thumbnails.medium" = Table.ExpandRecordColumn(#"Expanded items.snippet.thumbnails.default", "items.snippet.thumbnails.medium", {"url", "width", "height"}, {"items.snippet.thumbnails.medium.url", "items.snippet.thumbnails.medium.width", "items.snippet.thumbnails.medium.height"}),
    #"Expanded items.snippet.thumbnails.high" = Table.ExpandRecordColumn(#"Expanded items.snippet.thumbnails.medium", "items.snippet.thumbnails.high", {"url", "width", "height"}, {"items.snippet.thumbnails.high.url", "items.snippet.thumbnails.high.width", "items.snippet.thumbnails.high.height"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded items.snippet.thumbnails.high",{{"kind", type text}, {"etag", type text}, {"nextPageToken", type text}, {"regionCode", type text}, {"pageInfo.totalResults", Int64.Type}, {"pageInfo.resultsPerPage", Int64.Type}, {"items.kind", type text}, {"items.etag", type text}, {"items.id.kind", type text}, {"items.id.videoId", type text}, {"items.snippet.publishedAt", type datetime}, {"items.snippet.channelId", type text}, {"items.snippet.title", type text}, {"items.snippet.description", type text}, {"items.snippet.thumbnails.default.url", type text}, {"items.snippet.thumbnails.default.width", Int64.Type}, {"items.snippet.thumbnails.default.height", Int64.Type}, {"items.snippet.thumbnails.medium.url", type text}, {"items.snippet.thumbnails.medium.width", Int64.Type}, {"items.snippet.thumbnails.medium.height", Int64.Type}, {"items.snippet.thumbnails.high.url", type text}, {"items.snippet.thumbnails.high.width", Int64.Type}, {"items.snippet.thumbnails.high.height", Int64.Type}, {"items.snippet.channelTitle", type text}, {"items.snippet.liveBroadcastContent", type text}, {"items.snippet.publishTime", type datetime}}),
    #"Removed Columns1" = Table.RemoveColumns(#"Changed Type",{"items.snippet.thumbnails.medium.height", "items.snippet.thumbnails.medium.width", "items.snippet.thumbnails.default.height", "items.snippet.thumbnails.default.width", "items.snippet.description", "items.snippet.channelId", "items.id.kind", "items.kind", "regionCode", "pageInfo.totalResults", "pageInfo.resultsPerPage", "kind", "etag", "items.etag"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns1",{{"items.id.videoId", "videoId"}, {"items.snippet.publishedAt", "publishedAt"}, {"items.snippet.title", "Title"}, {"items.snippet.publishTime", "publishTime"}}),
    #"Inserted Time" = Table.AddColumn(#"Renamed Columns", "Time", each DateTime.Time([publishedAt]), type time),
    #"Inserted Start of Hour" = Table.AddColumn(#"Inserted Time", "Start of Hour", each Time.StartOfHour([Time]), type time),
    #"Extracted Date" = Table.TransformColumns(#"Inserted Start of Hour",{{"publishedAt", DateTime.Date, type date}}),
    #"Added Custom" = Table.AddColumn(#"Extracted Date", "videoURL", each "https://www.youtube.com/watch?v=" & [videoId]),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Custom",{"nextPageToken", "videoId", "publishedAt", "videoURL", "Start of Hour", "Time", "Title", "items.snippet.thumbnails.default.url", "items.snippet.thumbnails.medium.url", "items.snippet.thumbnails.high.url", "publishTime"}),
    #"Added Custom1" = Table.AddColumn(#"Reordered Columns", "video stats", each GetVideoStats([videoId])),
    #"Expanded video stats" = Table.ExpandTableColumn(#"Added Custom1", "video stats", {"Name", "Value"}, {"video stats.Name", "video stats.Value"}),
    #"Pivoted Column" = Table.Pivot(#"Expanded video stats", List.Distinct(#"Expanded video stats"[#"video stats.Name"]), "video stats.Name", "video stats.Value", List.Max),
    #"Changed Type1" = Table.TransformColumnTypes(#"Pivoted Column",{{"viewCount", Int64.Type}, {"likeCount", Int64.Type}, {"favoriteCount", Int64.Type}, {"commentCount", Int64.Type}}),
    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"items.snippet.thumbnails.default.url", "thumbnailsDefault"}, {"items.snippet.thumbnails.medium.url", "thumbnailsMedium"}, {"items.snippet.thumbnails.high.url", "thumbnailsHigh"}}),
    #"Added Custom2" = Table.AddColumn(#"Renamed Columns1", "Channel Name", each "Alex")
in
    #"Added Custom2"
```
### Transforming the channel statistics
```
let
    Source = Json.Document(Web.Contents(
        "https://www.googleapis.com/youtube/v3/channels", 
        [Query = [
            part = "statistics",
            id = "UC7cs8q-gJRlGwj4A8OmCmXg", // Replace with your channel ID
            key = "****************************************"
        ]]
    )),
    ChannelStats = Source[items]{0}[statistics],
    StatsTable = Record.ToTable(ChannelStats),
    #"Changed Type" = Table.TransformColumnTypes(StatsTable,{{"Value", Int64.Type}}),
    #"Pivoted Column" = Table.Pivot(#"Changed Type", List.Distinct(#"Changed Type"[Name]), "Name", "Value", List.Max),
    #"Renamed Columns" = Table.RenameColumns(#"Pivoted Column",{{"subscriberCount", "Subscribers"}, {"viewCount", "Views"}}),
    #"Removed Columns" = Table.RemoveColumns(#"Renamed Columns",{"hiddenSubscriberCount"}),
    #"Renamed Columns1" = Table.RenameColumns(#"Removed Columns",{{"videoCount", "Videos"}})
in
    #"Renamed Columns1"
```

##
The next step was combining ('append') the tables that contain videos from each channels. 
```
let
    Source = Table.Combine({#"Alex 1", #"Beast 1", #"Marques 1", #"Lawrence 1", #"Bello 1", #"Aina 1"}),
    #"Renamed Columns" = Table.RenameColumns(Source,{{"viewCount", "Views"}, {"likeCount", "Likes"}, {"commentCount", "Comments"}})
in
    #"Renamed Columns"
```
I then close and apply

## Wireframing and Design
I intended the dashboard to closely resemble the original Youtube interface, so I designed something like that in Figma. 

<img width="1915" height="967" alt="image" src="https://github.com/user-attachments/assets/473b75b6-7699-4923-8165-b5f8bd27aacb" />

<img width="1917" height="970" alt="image" src="https://github.com/user-attachments/assets/c2c69c4c-6174-49ad-9d85-7b629d3fbc9b" />

<img width="1917" height="968" alt="image" src="https://github.com/user-attachments/assets/3a5b6cc5-a35e-46d0-ac9a-96db879a6f22" />

## Power BI Dashboard
The dashboard in Power BI shows the 6 channels, the latest videos, and a button that redirects to where you can watch the videos. 

<img width="1911" height="905" alt="image" src="https://github.com/user-attachments/assets/d8461a8b-8157-4a25-a871-3c2e385df880" />

<img width="1916" height="907" alt="image" src="https://github.com/user-attachments/assets/aa8797f7-199d-4cea-8e36-b417e0c58fab" />


On the sidebar, navigtion to the channels has been included, and also by cicking on the channel icon, it takes you to a new page where you can see more details about the channels. 

<img width="1422" height="857" alt="image" src="https://github.com/user-attachments/assets/11483191-2194-47ae-bfe8-b6a3ebd0a859" />

## Publishing to Power BI Service
I used Power BI service to setup scheduled refreshes, so that the dashboard information stays up to date. I have to authenticate my connection using Google's Oauth. 
<img width="1083" height="658" alt="image" src="https://github.com/user-attachments/assets/8ee917ee-a185-452e-bab1-f76c22394d07" />

You can interact with the dashboard here: [Youtube Dashboard Project](https://app.fabric.microsoft.com/view?r=eyJrIjoiY2VkMTA2ZDAtMGE0Yy00YWYwLWJhMjQtMGRhMWZlZDk3OTk2IiwidCI6ImViODE5NmE3LTkyMGEtNDkxMS05M2ViLTdiMTYxYWI4ZDliOSJ9)

## Key Learnings
1. How to connect Power BI to an external API
2. Working with JSON data in Power Query
3. Structuring data models for multiple sources
4. Designing a clean, informative Power BI dashboard

Apart from this being an entirely fun project, it demonstrates how powerful and flexible **Power BI** can be when combined with the **YouTube Data API**. By connecting live API data to Power BI, I can monitor your favorite YouTube channels, analyze performance metrics, and visualize trends all in one place. Beyond personal use, this approach can be adapted for brands, creators, or marketing teams who want to track multiple channels, measure engagement, and make data-driven content decisions. It’s a great example of turning raw API data into meaningful visual insights. 


### Author
**James Oladejo**
📊 Data & BI Analyst | Product Analyst | BI Developer
📧 jmsola04@gmail.com
🌐 [Linkedin](https://linkedin.com/in/james-oladejo-2000/)
