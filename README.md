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
import base64

# Replace with your actual API key ID and secret
api_key_id = "your_api_key_id"
api_key_secret = "your_api_key_secret"

# Combine the API key ID and secret using a colon separator and encode it in base64
api_key_combined = f"{api_key_id}:{api_key_secret}"
api_key_encoded = base64.b64encode(api_key_combined.encode()).decode()

headers = {"Authorization": f"Basic {api_key_encoded}"}

# Example API request: Get video instances by user
url = "https://studio.bhuman.ai/api/video_instance"
response = requests.get(url, headers=headers)
print(response.json())
```

Replace `your_api_key` with the actual API key you generated earlier.

## Understanding URLs

To use our API effectively, you'll need to understand how to extract information from the platform's URLs. Here are two example URLs:

- Campaign ID URL: `https://app.dev.bhuman.ai/campaign/generatedVideos/b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Instance ID URL: `https://app.dev.bhuman.ai/instance/7d859e18-dc89-446f-9718-9533bb178a75`

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

**Endpoint:** `https://studio.bhuman.ai/swagger-ui/#/%22video_instance%22/get_video_instance`

**Method:** `GET`

**Description:** Retrieves video instances associated with the authenticated user.

### Get video instance by instance ID

**Endpoint:** `https://studio.bhuman.ai/swagger-ui/#/%22video_instance%22/get_video_instance`

**Method:** `GET`

**Description:** Retrieves a specific video instance based on the provided instance ID.

### Generate video by instance ID

**Endpoint:** `https://studio.bhuman.ai/swagger-ui/#/%22pipeline%22/process_try_sample`

**Method:** `POST`

**Description:** Generates a video using the provided instance ID.

### Get generated videos by instance ID

**Endpoint:** `https://studio.bhuman.ai/swagger-ui/#/%22generated_video%22/get_generated_video_by_video_instance_id`

**Method:** `GET`

**Description:** Retrieves generated videos associated with the provided instance ID.

### Get workspace

**Endpoint:** `https://user.bhuman.ai/swagger-ui/#/%22workspace%22/get_workspace`

**Method:** `GET`

**Description:** Retrieves information about the authenticated user's workspace.

### Get products from store

**Endpoint:** `https://store.bhuman.ai/swagger-ui/#/%22store%22/get_products`

**Method:** `GET`

**Description:** Retrieves available products from the store.

### Use product

**Endpoint:** `https://store.bhuman.ai/swagger-ui/#/%22store%22/use_product`

**Method:** `POST`

**Description:** Uses a specific product from the store.

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
url = "https://studio.bhuman.ai/api/ai_studio/video_instance"
response = requests.get(url, headers=headers)
print(response.json())

# Get video instance by instance ID
instance_id = "7d859e18-dc89-446f-9718-9533bb178a75"
url = f"https://studio.bhuman.ai/api/ai_studio/video_instance/{instance_id}"
response = requests.get(url, headers=headers)
print(response.json())

# Generate video by instance ID
url = "https://studio.bhuman.ai/api/ai_studio/try_sample"
payload = {
  "assets": null,
  "callback_url": null,
  "campaign_result_ids": null,
  "filter_complex": null,
  "names": [],
  "variables": [],
  "video_instance_id": "00000000-0000-0000-0000-000000000000"
}
response = requests.post(url, json=payload, headers=headers)
print(response.json())

# Get generated videos by instance ID
url = f"https://studio.bhuman.ai/api/ai_studio/generated_video_by_video_instance_id/{instance_id}"
response = requests.get(url, headers=headers)
print(response.json())

# Get workspace
url = "https://user.bhuman.ai/api/workspace"
response = requests.get(url, headers=headers)
print(response.json())

# Get products from store
url = "https://store.bhuman.ai/api/store/products"
response = requests.get(url, headers=headers)
print(response.json())

# Use product
product_id = "example_product_id"
url = "https://store.bhuman.ai/api/store/use_product"
payload = {
  "folder_id": null,
  "id": "00000000-0000-0000-0000-000000000000",
  "video_instance_id": null,
  "workspace_id": "00000000-0000-0000-0000-000000000000"
}
response = requests.post(url, json=payload, headers=headers)
print(response.json())
```

Replace `your_api_key_id` and `your_api_key_secret`  with your actual API id and secret, and replace `instance_id` and `campaign_id` with valid instance and campaign IDs.

## Support

If you have any questions or need further assistance, please don't hesitate to contact our support team at help@bhuman.ai. We're here to help!