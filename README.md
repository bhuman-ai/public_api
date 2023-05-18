# BHuman Video Generation API Guide

Welcome to BHuman's Video Generation API Guide, a powerful tool that allows you to generate customized videos using AI. With our API, you can easily integrate video generation capabilities into your applications and create stunning, personalized content for your users. In this README, we'll provide you with step-by-step instructions on how to use our API.

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
import base64

# Replace with your actual API key ID and secret
api_key_id = "your_api_key_id"
api_key_secret = "your_api_key_secret"

# Combine the API key ID and secret using a colon separator and encode it in base64
api_key_combined = f"{api_key_id}:{api_key_secret}"
api_key_encoded = base64.b64encode(api_key_combined.encode()).decode()

headers = {"Authorization": f"Basic {api_key_encoded}"}

# Example API request: Get video instances by user
url = "https://studio.bhuman.ai/api/ai_studio/video_instance"
response = requests.get(url, headers=headers)
print(response.json())
```

Replace `your_api_key` with the actual API key you generated earlier.

## Understanding URLs

To use our API effectively, you'll need to understand how to extract information from the platform's URLs. Here are two example URLs:

- Campaign ID URL: `https://app.bhuman.ai/campaign/generatedVideos/b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Instance ID URL: `https://app.bhuman.ai/instance/7d859e18-dc89-446f-9718-9533bb178a75`

The last part of the URL (after the last `/`) represents the respective ID:

- Campaign ID: `b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Instance ID: `7d859e18-dc89-446f-9718-9533bb178a75`

## API Endpoints

Here's a list of available API endpoints:

1. [Get video instances by user](#get-video-instances-by-user)
2. [Get video instance by instance ID](#get-video-instance-by-instance-id)
3. [Generate video by instance ID](#generate-video-by-instance-id)
4. [Get generated videos by instance ID](#get-generated-videos-by-instance-id)
5. [Get generated video by generate ID](#get-generated-video-by-generate-id)
6. [Get workspace](#get-workspace)
7. [Get products from store](#get-products-from-store)
8. [Use product](#use-product)
9. [Get categories and tags](#get-categories-and-tags)

### Get video instances by user

**Endpoint:** `https://studio.bhuman.ai/api/ai_studio/video_instances`

**Method:** `GET`

**Parameters:**

```
page: Integer - Required (start from 0)
size: Integer - Required (max size is 100)
```

**Description:** Retrieves video instances associated with the authenticated user.

**Reponse:** Video Instance list and total count

```
{
  total: Int,
  instances: [
    {
      id: Uuid,
      name: String,
      user_id: String,
      folder_id: Uuid,
      actor_id: Uuid - Nullable,
      product_id: Uuid - Nullable,
      industry: Int,
      use_case: String - Nullable,
      created_at: Date,
      updated_at: Date
    }
  ]
}
```

### Get video instance by instance ID

**Endpoint:** `https://studio.bhuman.ai/api/ai_studio/video_instance`

**Method:** `GET`

**Parameters:** 

```
id: Uuid - Required  
```

**Description:** Retrieves a specific video instance based on the provided instance ID.

**Reponse:** Video Instance list

```
[
  {
    id: Uuid,
    name: String,
    user_id: String,
    folder_id: Uuid,
    actor_id: Uuid - Nullable,
    product_id: Uuid - Nullable,
    industry: Int,
    use_case: String - Nullable,
    created_at: Date,
    updated_at: Date
  }
]
```

### Generate video by instance ID

**Endpoint:** `https://studio.bhuman.ai/api/ai_studio/try_sample`

**Method:** `POST`

**Parameters:** 

```
callback_url: String - Optional,
names: String[][] - Required,
variables: String[] - Required,
video_instance_id: Uuid - Required
```

**Description:** Generates a video using the provided instance ID.

If `callback_url` is empty, you may get the generation results using [Get generated videos by instance ID](#get-generated-videos-by-instance-id)

**Reponse:** Video generation id list, this will return as soon as called API

```
{
  "result": [
    "c10ee155-6202-4cec-9a40-dde536e2ab4e"
  ]
}
```

After finishing video generation, it will return the results via `callback_url` using the `POST` method, and here is the payload.

(1 callback per 1 generation)

```
id": Uuid,                <- generation id = c10ee155-6202-4cec-9a40-dde536e2ab4e
status": String,          <- generation status = 'succeeded', 'failed'
video_url": String,       <- vimeo url = https://vimeo.com/822632664/8cbe80ad5f
thumbnail": String,       <- thumbnail url = https://tmp-prod-492171.s3.us-east-2.amazonaws.com/pipeline-out/dc44f89e-cc94-48a0-8261-8013081eba0c.png
url": String,             <- downloadable url = https://tmp-prod-492171.s3.us-east-2.amazonaws.com/pipeline-out/0515c90c-44ae-4755-96c7-843040c3a86f.mp4
```

### Get generated videos by instance ID

**Endpoint:** `https://studio.bhuman.ai/api/ai_studio/generated_video_by_video_instance_id`

**Method:** `GET`

**Parameters:**

```
video_instance_id: Uuid - Required
page: Integer - Optional (start from 0)
size: Integer - Optional (default size is 100)
```

**Description:** Retrieves generated videos associated with the provided instance ID.

If `page` and `size` are empty, it will return the first 100 generation results.

**Response:** Generated Videos

```
[
  {
      user_id: String,
      video_id: Uuid,
      actor_id: Uuid,
      video_instance_id: Uuid - Nullable,
      campaign_id: Uuid - Nullable,
      url: String - Nullable,
      vimeo_url: String - Nullable,
      thumbnail: String - Nullable,
      status: String,
      checksum: Int,
      execution_name: String - Nullable, (If video generation failed, you can report using this execution id.)
      row_index: Int - Nullable, (It's the same number as names value sequence that you request in the Generate Video API.)
      text: String[] - Nullable, (["Don", "Apple"])
      message: String - Nullable, 
      created_at: Date,
      updated_at: Date,
  }
]
```

### Get generated video by generate ID

**Endpoint:** `https://studio.bhuman.ai/api/ai_studio/generated_video_by_id`

**Method:** `GET`

**Parameters:**

```
id: Uuid - Required
```

**Description:** Retrieves generated video associated with the provided generate ID that returned by `try_sample`.

**Response:** Generated Video

```
{
      user_id: String,
      video_id: Uuid,
      actor_id: Uuid,
      video_instance_id: Uuid - Nullable,
      campaign_id: Uuid - Nullable,
      url: String - Nullable,
      vimeo_url: String - Nullable,
      thumbnail: String - Nullable,
      status: String,
      checksum: Int,
      execution_name: String - Nullable, (If video generation failed, you can report using this execution id.)
      row_index: Int - Nullable, (It's the same number as names value sequence that you request in the Generate Video API.)
      text: String[] - Nullable, (["Don", "Apple"])
      message: String - Nullable, 
      created_at: Date,
      updated_at: Date,
}
```

### Get workspace

**Endpoint:** `https://user.bhuman.ai/api/workspace`

**Method:** `GET`

**Parameters:** 

```
id: Uuid - Optional  
```

**Description:** Retrieves information about the authenticated user's workspace.

**Response:** Workspace list

```
[
  {
    workspace_id: Uuid,          // workspace id
    user_id: String,             // user id
    name: String,                // workspace name
    role: String,                // workspace role, owner, member
    description: String,         // workspace description
    created_at: Date,
    updated_at: Date,
  }
]
```

### Get products from store

**Endpoint:** `https://store.bhuman.ai/api/store/product`

**Method:** `GET`

**Description:** Retrieves available products from the store.

**Response:** Product list

```
[
  {
    id: Uuid,
    name: String,
    category: Int,
    description: String - Nullable,
    video_url: String - Nullable,
    image_url: String - Nullable,
    thumbnail: String - Nullable,
    creator_id: String,
    actor_id: Uuid,
    length: Int,
    downloads: Int,
    ratings: Float,
    reviews: Int,
    tags: Int[] - Nullable,
    created_at: Date,
  }
]
```

### Use product

**Endpoint:** `https://store.bhuman.ai/api/store/product/use`

**Method:** `POST`

**Parameters:**

```
id: Uuid - Required,
workspace_id: Uuid - Required,
folder_id: Uuid - Optional,
video_instance_id: Uuid - Optional,
```

**Description:** Uses a specific product from the store.

If `folder_id` and `video_instance_id` are empty, it will create a new folder and video instance.

If `video_instance_id` is present, `folder_id` can not be empty.

If `folder_id` is present and `video_instance_id` is empty, it will create a new video instance.

**Reponse:** 

```
{
    video_instance_id: Uuid,
    video_id: Uuid,
    folder_id: Uuid,
    product: {
      id: Uuid,
      name: String,
      category: Int,
      description: String - Nullable,
      video_url: String - Nullable,
      image_url: String - Nullable,
      thumbnail: String - Nullable,
      creator_id: String,
      actor_id: Uuid,
      length: Int,
      downloads: Int,
      ratings: Float,
      reviews: Int,
      tags: Int[] - Nullable,
      created_at: Date,
    },
    segments: Uuid[],
}
```

### Get categories and tags

**Endpoint:** `https://store.bhuman.ai/api/store/settings`

**Method:** `GET`

**Description:** Get categories and tags from the store.

**Response:**

```
{
  "category": [
    {
      "id": Int,
      "name": String
    }
  ],
  "tags": [
    {
      "id": Int,
      "name": String
    }
  ]
}
```

## Examples

Here are some examples of how to use the API with Python and the `requests` library:

```python
import requests
import base64

# Replace with your actual API key ID and secret
api_key_id = "your_api_key_id"
api_key_secret = "your_api_key_secret"

# Combine the API key ID and secret using a colon separator and encode it in base64
api_key_combined = f"{api_key_id}:{api_key_secret}"
api_key_encoded = base64.b64encode(api_key_combined.encode()).decode()

headers = {"Authorization": f"Basic {api_key_encoded}"}

# Get video instances by user
url = "https://studio.bhuman.ai/api/ai_studio/video_instances?page=0&size=100"
response = requests.get(url, headers=headers)
print(response.json())

# Get video instance by instance ID
instance_id = "7d859e18-dc89-446f-9718-9533bb178a75"
url = f"https://studio.bhuman.ai/api/ai_studio/video_instance?id={instance_id}"
response = requests.get(url, headers=headers)
print(response.json())

# Generate video by instance ID
url = "https://studio.bhuman.ai/api/ai_studio/try_sample"
payload = {
  "callback_url": "https://xxx.yyy.zzz?my_query_param1=aaa&my_query_param2=bbb",
  "names": [["Don", "Apple"], ["James", "Google"]],
  "variables": ["name", "company"],
  "video_instance_id": instance_id
}
response = requests.post(url, json=payload, headers=headers)
print(response.json())

# Get generated videos by instance ID
url = f"https://studio.bhuman.ai/api/ai_studio/generated_video_by_video_instance_id?video_instance_id={instance_id}&page=0&size=100"
response = requests.get(url, headers=headers)
print(response.json())

# Get workspace
url = "https://user.bhuman.ai/api/workspace"
response = requests.get(url, headers=headers)
print(response.json())

# Get products from store
url = "https://store.bhuman.ai/api/store/product"
response = requests.get(url, headers=headers)
print(response.json())

# Use product
product_id = "example_product_id"
workspace_id = "example_workspace_id"
url = "https://store.bhuman.ai/api/store/product/use"
payload = {
  "folder_id": null,
  "id": product_id,
  "video_instance_id": null,
  "workspace_id": workspace_id
}
response = requests.post(url, json=payload, headers=headers)
print(response.json())

# Get categories and tags from store
url = "https://store.bhuman.ai/api/store/settings"
response = requests.get(url, headers=headers)
print(response.json())
```

Replace `your_api_key_id` and `your_api_key_secret`  with your actual API id and secret, and replace `instance_id` and `campaign_id` with valid instance and campaign IDs.

## Support

If you have any questions or need further assistance, please don't hesitate to contact our support team at help@bhuman.ai. We're here to help!
