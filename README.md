# BHuman Video Generation API

Welcome to BHuman's Video Generation API, a powerful tool that allows you to generate customized videos using AI. With our API, you can easily integrate video generation capabilities into your applications and create stunning, personalized content for your users. In this README, we'll provide you with step-by-step instructions on how to use our API.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Authentication](#authentication)
- [Understanding URLs](#understanding-urls)
- [API Endpoints](#api-endpoints)
- [Examples](#examples)
- [Support](#support)

## Prerequisites

To use our API, you'll need to generate an API key. Follow these steps:

1. Log in to your BHuman account.
2. Navigate to the "Settings" page.
3. Click on "API keys."
4. Click the "Generate" button.
5. Copy the generated API key.

Keep your API key safe, as you'll need it for authentication.

## Authentication

Our API uses Basic Authentication. You'll need to include your API key in the `Authorization` header of your requests. Here's an example of how to do this with Python:

```python
import requests

api_key = "your_api_key"
headers = {"Authorization": f"Basic {api_key}"}
```

Replace `your_api_key` with the actual API key you generated earlier.

## Understanding URLs

To use our API effectively, you'll need to understand how to extract information from the platform's URLs. Here are two example URLs:

- Campaign ID URL: `https://app.dev.BHuman.ai/campaign/generatedVideos/b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Instance ID URL: `https://app.dev.BHuman.ai/instance/7d859e18-dc89-446f-9718-9533bb178a75`

The last part of the URL (after the last `/`) represents the respective ID:

- Campaign ID: `b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Instance ID: `7d859e18-dc89-446f-9718-9533bb178a75`

## API Endpoints

Here's a list of available API endpoints:

1. [Get video instances by user](#get-video-instances-by-user)
2. [Get video instance by instance ID](#get-video-instance-by-instance-id)
3. [Generate video by instance ID](#generate-video-by-instance-id)
4. [Get generated videos by instance ID](#get-generated-videos-by-instance-id)
5. [Get workspace](#get-workspace)
6. [Get products from store](#get-products-from-store)
7. [Use product](#use-product)

### Get video instances by user

**Endpoint:** `https://studio.BHuman.ai/swagger-ui/#/%22video_instance%22/get_video_instance`

**Method:** `GET`

**Description:** Retrieves video instances associated with the authenticated user.

### Get video instance by instance ID

**Endpoint:** `https://studio.BHuman.ai/swagger-ui/#/%22video_instance%22/get_video_instance`

**Method:** `GET`

**Description:** Retrieves a specific video instance based on the provided instance ID.

### Generate video by instance ID

**Endpoint:** `https://studio.BHuman.ai/swagger-ui/#/%22pipeline%22/process_try_sample`

**Method:** `POST`

**Description:** Generates a video using the provided instance ID.

### Get generated videos by instance ID

**Endpoint:** `https://studio.BHuman.ai/swagger-ui