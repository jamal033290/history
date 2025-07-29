import streamlit as st
import requests
from datetime import datetime, timedelta

# YouTube API Key
API_KEY = "AIzaSyAvFafNFyNd9qIJLkuhnykHU_TbK0Tm-mk"
YOUTUBE_SEARCH_URL = "https://www.googleapis.com/youtube/v3/search"
YOUTUBE_VIDEO_URL = "https://www.googleapis.com/youtube/v3/videos"
YOUTUBE_CHANNEL_URL = "https://www.googleapis.com/youtube/v3/channels"

# Streamlit App Title
st.title("YouTube Viral Topics Tool (Videos Longer Than 8 Minutes)")

# Input Fields
days = st.number_input("Enter Days to Search (1-30):", min_value=1, max_value=30, value=5)

# Keywords
keywords = [
"History Facts Today", "Ancient History Secrets", "World War II Stories", 
"World History Facts", "History Untold Truth", "Ancient Civilizations History", 
"Medieval History Explained", "Historical Events Explained", "Famous Historical Figures", 
"History Timeline Facts", "History of Ancient Empires", "Modern History Events", 
"History Mysteries Solved", "History That Changed The World", "Unknown History Facts", 
"History of Kings and Queens", "Great Battles in History", "History Lessons for Today", 
"Dark History Secrets", "Historical Stories That Shocked", "History Documentaries Explained", 
"History Events That Shaped The World", "History Legends and Myths", "Ancient History Discoveries", 
"World History Timeline Explained", "History of
]

# Function to convert ISO 8601 duration to seconds
def parse_duration(duration):
    hours = minutes = seconds = 0
    duration = duration.replace("PT", "")
    if "H" in duration:
        hours, duration = duration.split("H")
        hours = int(hours)
    if "M" in duration:
        minutes, duration = duration.split("M")
        minutes = int(minutes)
    if "S" in duration:
        seconds = int(duration.replace("S", ""))
    return hours * 3600 + minutes * 60 + seconds

if st.button("Fetch Data"):
    try:
        start_date = (datetime.utcnow() - timedelta(days=int(days))).isoformat("T") + "Z"
        all_results = []

        for keyword in keywords:
            st.write(f"Searching for keyword: {keyword}")

            search_params = {
                "part": "snippet",
                "q": keyword,
                "type": "video",
                "order": "viewCount",
                "publishedAfter": start_date,
                "maxResults": 5,
                "key": API_KEY,
            }

            response = requests.get(YOUTUBE_SEARCH_URL, params=search_params)
            data = response.json()

            if "items" not in data or not data["items"]:
                continue

            videos = data["items"]
            video_ids = [video["id"]["videoId"] for video in videos]
            channel_ids = [video["snippet"]["channelId"] for video in videos]

            if not video_ids or not channel_ids:
                continue

            # Fetch video details (including duration)
            stats_params = {"part": "contentDetails,statistics", "id": ",".join(video_ids), "key": API_KEY}
            stats_response = requests.get(YOUTUBE_VIDEO_URL, params=stats_params)
            stats_data = stats_response.json()

            channel_params = {"part": "statistics", "id": ",".join(channel_ids), "key": API_KEY}
            channel_response = requests.get(YOUTUBE_CHANNEL_URL, params=channel_params)
            channel_data = channel_response.json()

            if "items" not in stats_data or "items" not in channel_data:
                continue

            stats = stats_data["items"]
            channels = channel_data["items"]

            for video, stat, channel in zip(videos, stats, channels):
                duration = stat["contentDetails"]["duration"]
                duration_seconds = parse_duration(duration)

                # ✅ Only include videos longer than 8 minutes (480 seconds)
                if duration_seconds >= 480:
                    title = video["snippet"].get("title", "N/A")
                    description = video["snippet"].get("description", "")[:200]
                    video_url = f"https://www.youtube.com/watch?v={video['id']['videoId']}"
                    views = int(stat["statistics"].get("viewCount", 0))
                    subs = channel["statistics"].get("subscriberCount", "Hidden")

# Only filter if it's not hidden and < 3000
if subs != "Hidden" and int(subs) < 3000:
    all_results.append({
        "Title": title,
        "Description": description,
        "URL": video_url,
        "Views": views,
        "Subscribers": subs,
        "Duration": duration
    })



        if all_results:
            st.success(f"Found {len(all_results)} results (videos longer than 8 minutes).")
            for result in all_results:
                st.markdown(
                    f"**Title:** {result['Title']}  \n"
                    f"**Description:** {result['Description']}  \n"
                    f"**URL:** [Watch Video]({result['URL']})  \n"
                    f"**Views:** {result['Views']}  \n"
                    f"**Subscribers:** {result['Subscribers']}  \n"
                    f"**Duration:** {result['Duration']}"
                )
                st.write("---")
        else:
            st.warning("No videos longer than 8 minutes found for channels under 3,000 subscribers.")

    except Exception as e:
        st.error(f"An error occurred: {e}")
