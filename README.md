# TikTok
import requests
import json
from datetime import datetime, timedelta
import time
import csv
import pandas as pd
import sys

class TikTokAPI:
    def __init__(self, client_key, client_secret):
        access_token = self.get_access_token(client_key, client_secret)
        if access_token is None:
            print(f"Error: access token is none, exiting")
            sys.exit()
        self.base_url = "https://open.tiktokapis.com/v2/"
        self.headers = {
            "Authorization": f"Bearer {self.access_token}",
            "Content-Type": "application/json"
        }

    def get_access_token(self, client_key, client_secret):
        token_url = 'https://open.tiktokapis.com/v2/oauth/token/'

        data = {
            'client_key': client_key,
            'client_secret': client_secret,
            'grant_type': 'client_credentials'
        }

        headers = {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Cache-Control': 'no-cache'
        }

        response = requests.post(token_url, data=data, headers=headers)
        if response.status_code == 200:
            access_token = response.json().get('data', {}).get('access_token')
            print(f"Access Token: {access_token}")
            return access_token
        else:
            print(f"Error: {response.status_code} - {response.text}")
            return None

    def search_videos(self, hashtags, start_date, end_date, max_results=100):
        endpoint = "research/video/query/"
        data = {
            "hashtag_names": hashtags,
            "start_date": start_date.strftime('%Y-%m-%d'),
            "end_date": end_date.strftime('%Y-%m-%d'),
            "max_count": max_results
        }
        return self._make_request(endpoint, data)

    def search_video_comments(self, video_id, max_results=100):
        endpoint = "research/video/comment/list/"
        data = {
            "video_id": video_id,
            "max_count": max_results
        }
        return self._make_request(endpoint, data)

    def _make_request(self, endpoint, data):
        url = self.base_url + endpoint
        response = requests.post(url, data=data)
        if response.status_code == 200:
            return response.json()
        else:
            print(f"Error: {response.status_code}")
            print(f"Response: {response.text}")
            return None

def process_video_data(video):
    return {
        "id": video.get("id"),
        "create_time": video.get("create_time"),
        "username": video.get("username"),
        "region_code": video.get("region_code"),
        "video_description": video.get("video_description"),
        "music_id": video.get("music_id"),
        "like_count": video.get("like_count"),
        "comment_count": video.get("comment_count"),
        "share_count": video.get("share_count"),
        "view_count": video.get("view_count"),
        "effect_ids": ''.join(video.get("effect_ids")),
        "hashtag_names": ''.join(video.get("hashtag_names")),
        "video_mention_list": ''.join(video.get("video_mention_list")),
        "playlist_id": video.get("playlist_id"),
        "voice_to_text": video.get("voice_to_text"),
        "video_duration": video.get("video_duration"),
        "favourites_count": video.get("favourites_count"),
    }

def process_video_comment_data(comment):
    return {
        "id": comment.get("id"),
        "text": comment.get("text"),
        "video_id": comment.get("video_id"),
        "like_count": comment.get("like_count"),
        "reply_count": comment.get("reply_count"),
        "create_time": comment.get("create_time")
    }

def save_to_csv(data, filename):
    with open(filename, 'a', newline='', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=data[0].keys())
        writer.writeheader()
        writer.writerows(data)

def main():
    client_key = input("Please enter your TikTok client key, it can be found in your TikTok application page: ")
    client_secret = input("Please enter your TikTok client secret, it can be found in your TikTok application page: ")

    api = TikTokAPI(client_key, client_secret)

    hashtags = input("Enter search hashtags (comma-separated): ").split(',')
    start_date_str = input("Enter start date (YYYY-MM-DD): ")
    end_date_str = input("Enter end date (YYYY-MM-DD): ")
    
    start_date = datetime.strptime(start_date_str, '%Y-%m-%d')
    end_date = datetime.strptime(end_date_str, '%Y-%m-%d')

    all_videos = []

    videos_filename = f"tiktok_videos_{start_date_str}_to_{end_date_str}.csv"
    existing_videos_df = pd.read_csv(videos_filename)

    for hashtag in hashtags:
        print(f"Searching for videos with hashtag: {hashtag}")
        results = api.search_videos(hashtag.strip(), start_date, end_date)

        if results.id in existing_videos_df["id"].values:
            continue
        
        if results and 'videos' in results:
            videos = results['videos']
            all_videos.extend([process_video_data(video) for video in videos])
            print(f"Found {len(videos)} videos for hashtag '{hashtag}'")
        else:
            print(f"No results found for keyword '{keyword}'")

        if len(all_videos) >= 1000:
            print(f"We have stored 1000 videos, pausing now")
            break

        time.sleep(1)  # 添加延迟以避免超过API速率限制

    if all_videos:
        save_to_csv(all_videos, videos_filename)
        print(f"Data saved to {videos_filename}")
    else:
        print("No videos found for the given criteria.")

    comments_filename = f"tiktok_comments_{start_date_str}_to_{end_date_str}.csv"
    all_comments = []
    for video in all_videos:
        video_id = video.get("id")
        comments_results = api.search_video_comments(video_id)

        if comments_results and 'comments' in comments_results:
            comments = comments_results['comments']
            all_comments.extend([process_video_comment_data(comment) for comment in comments])
            print(f"Found {len(comments)} comments for video '{video_id}'")
        else:
            print(f"No comments found for video '{video_id}'")

    if all_comments:
        save_to_csv(all_comments, comments_filename)
        print(f"Data saved to {comments_filename}")
    else:
        print("No videos and comments found for the given criteria.")

if __name__ == "__main__":
    main()



[Download Verification File](tiktokqtv8IpXVak6nUwfiBaoJQTqsNnCMys1b.txt)
