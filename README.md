# Chat API Documentation

[![Laravel](https://img.shields.io/badge/Laravel-10.x-red.svg)](https://laravel.com)
[![PHP](https://img.shields.io/badge/PHP-8.1+-blue.svg)](https://php.net)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![API](https://img.shields.io/badge/API-REST-orange.svg)](https://restfulapi.net)

> A comprehensive real-time chat API built with Laravel, featuring private/group conversations, media sharing, typing indicators, and online status tracking.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Authentication](#authentication)
- [Base URL](#base-url)
- [API Endpoints](#api-endpoints)
  - [1. Conversation Management](#1-conversation-management)
  - [2. Message Management](#2-message-management)
  - [3. Media Management](#3-media-management)
  - [4. Typing Indicators](#4-typing-indicators)
  - [5. Online Status Management](#5-online-status-management)
- [Real-Time Events](#real-time-events)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [File Upload Guidelines](#file-upload-guidelines)
- [Implementation Notes](#implementation-notes)
- [Frontend Integration](#frontend-integration)
- [Contributing](#contributing)
- [License](#license)

## 🚀 Overview

The Chat API provides a comprehensive real-time messaging system with support for both private and group conversations. The system includes features like media sharing, typing indicators, online status tracking, and real-time notifications.

### 🎯 Features

- ✅ **Private & Group Conversations** - Support for both one-on-one and group chats
- ✅ **Real-Time Messaging** - Instant message delivery with Laravel Broadcasting
- ✅ **Media Sharing** - Upload and share images, videos, audio, and documents
- ✅ **Typing Indicators** - Real-time typing status notifications
- ✅ **Online Status** - Track user presence with heartbeat system
- ✅ **S3 Integration** - Secure file storage with AWS S3
- ✅ **Pagination** - Efficient message loading with pagination
- ✅ **Authentication** - Secure API access with Laravel Sanctum
- ✅ **Error Handling** - Comprehensive error responses
- ✅ **Rate Limiting** - Protection against API abuse

## 🔐 Authentication

All chat endpoints require authentication using Laravel Sanctum. Include the bearer token in the Authorization header:

```http
Authorization: Bearer {your_token}
```

## 🌐 Base URL

```
/api/chats
```

---

## 📡 API Endpoints

### 1. Conversation Management

#### 1.1 Get All Conversations

**Endpoint:** `GET /api/chats/conversations`

**Description:** Retrieves all conversations for the authenticated user, including the latest message and participant information.

**Response:**
```json
{
  "success": true,
  "message": "Conversations retrieved successfully",
  "data": [
    {
      "id": 1,
      "type": "private",
      "name": null,
      "created_at": "2024-01-01T00:00:00.000000Z",
      "updated_at": "2024-01-01T12:00:00.000000Z",
      "users": [
        {
          "id": 1,
          "name": "John Doe",
          "email": "john@example.com",
          "avatar_url": "https://example.com/avatar.jpg"
        }
      ],
      "last_message": {
        "id": 10,
        "body": "Hello there!",
        "created_at": "2024-01-01T12:00:00.000000Z",
        "sender": {
          "id": 2,
          "name": "Jane Smith"
        },
        "media": []
      }
    }
  ]
}
```

#### 1.2 Create New Conversation

**Endpoint:** `POST /api/chats/conversations`

**Description:** Creates a new private or group conversation. For private conversations, if a conversation already exists between the specified users, it returns the existing conversation.

**Request Body:**
```json
{
  "type": "private|group",
  "user_ids": [2, 3, 4],
  "name": "Group Name"
}
```

**Validation Rules:**
- `type`: Required, must be either "private" or "group"
- `user_ids`: Required array with at least 1 user ID
- `user_ids.*`: Must exist in users table
- `name`: Optional string (recommended for group conversations)

**Response:**
```json
{
  "success": true,
  "message": "Conversation created successfully",
  "data": {
    "id": 1,
    "type": "group",
    "name": "My Group",
    "created_at": "2024-01-01T00:00:00.000000Z",
    "updated_at": "2024-01-01T00:00:00.000000Z",
    "users": [
      {
        "id": 1,
        "name": "John Doe"
      }
    ]
  }
}
```

#### 1.3 Get Specific Conversation

**Endpoint:** `GET /api/chats/conversations/{id}`

**Description:** Retrieves a specific conversation with all its messages and participants.

**Response:**
```json
{
  "success": true,
  "message": "Conversation retrieved successfully",
  "data": {
    "id": 1,
    "type": "private",
    "name": null,
    "created_at": "2024-01-01T00:00:00.000000Z",
    "updated_at": "2024-01-01T12:00:00.000000Z",
    "users": [
      {
        "id": 1,
        "name": "John Doe"
      },
      {
        "id": 2,
        "name": "Jane Smith"
      }
    ],
    "messages": [
      {
        "id": 1,
        "body": "Hello!",
        "created_at": "2024-01-01T10:00:00.000000Z",
        "sender": {
          "id": 1,
          "name": "John Doe"
        },
        "media": []
      }
    ]
  }
}
```

#### 1.4 Rename Group Conversation

**Endpoint:** `PUT /api/chats/conversations/{id}/rename`

**Description:** Renames a group conversation. Only works for group conversations.

**Request Body:**
```json
{
  "name": "New Group Name"
}
```

**Validation Rules:**
- `name`: Required string

**Response:**
```json
{
  "success": true,
  "message": "Group name updated",
  "data": {
    "id": 1,
    "type": "group",
    "name": "New Group Name",
    "updated_at": "2024-01-01T12:00:00.000000Z"
  }
}
```

#### 1.5 Add Members to Group

**Endpoint:** `POST /api/chats/conversations/{id}/members`

**Description:** Adds new members to a group conversation. Only works for group conversations.

**Request Body:**
```json
{
  "user_ids": [5, 6, 7]
}
```

**Validation Rules:**
- `user_ids`: Required array
- `user_ids.*`: Must exist in users table

**Response:**
```json
{
  "success": true,
  "message": "Users added to group",
  "data": {
    "id": 1,
    "type": "group",
    "name": "My Group",
    "users": [
      {
        "id": 1,
        "name": "John Doe"
      },
      {
        "id": 5,
        "name": "New Member"
      }
    ]
  }
}
```

#### 1.6 Leave Conversation

**Endpoint:** `DELETE /api/chats/conversations/{id}/leave`

**Description:** Allows the authenticated user to leave a conversation.

**Response:**
```json
{
  "success": true,
  "message": "You have left the conversation"
}
```

### 2. Message Management

#### 2.1 Get Conversation Messages

**Endpoint:** `GET /api/chats/conversations/{id}/messages`

**Description:** Retrieves paginated messages from a specific conversation, ordered by creation date (newest first).

**Query Parameters:**
- `page`: Page number (default: 1)
- `per_page`: Messages per page (default: 50)

**Response:**
```json
{
  "success": true,
  "message": "Messages retrieved successfully",
  "data": {
    "current_page": 1,
    "data": [
      {
        "id": 1,
        "body": "Hello there!",
        "created_at": "2024-01-01T12:00:00.000000Z",
        "sender": {
          "id": 1,
          "name": "John Doe",
          "avatar_url": "https://example.com/avatar.jpg"
        },
        "media": [
          {
            "id": 1,
            "file_name": "image.jpg",
            "file_url": "https://s3.amazonaws.com/bucket/image.jpg",
            "file_type": "image/jpeg",
            "file_size": 1024000
          }
        ]
      }
    ],
    "total": 150,
    "per_page": 50,
    "last_page": 3
  }
}
```

#### 2.2 Send Message

**Endpoint:** `POST /api/chats/messages`

**Description:** Sends a new message to a conversation. Supports text and media attachments.

**Request Body:**
```json
{
  "conversation_id": 1,
  "body": "Hello everyone!",
  "media": []
}
```

**Validation Rules:**
- `conversation_id`: Required, must exist in conversations table
- `body`: Optional string
- `media`: Optional array of files
- `media.*`: File, max 100MB per file

**Features:**
- Supports multiple media files
- Automatically processes media through S3 service
- Sends real-time notifications to conversation participants
- Broadcasts message events for real-time updates

**Response:**
```json
{
  "success": true,
  "message": "Message sent successfully",
  "data": {
    "id": 1,
    "body": "Hello everyone!",
    "created_at": "2024-01-01T12:00:00.000000Z",
    "sender": {
      "id": 1,
      "name": "John Doe"
    },
    "media": []
  }
}
```

#### 2.3 Get Latest Messages (Polling)

**Endpoint:** `GET /api/chats/conversations/poll/{conversationId}/{lastMessageId?}`

**Description:** Retrieves new messages since a specific message ID. Used for real-time message polling.

**Parameters:**
- `conversationId`: Required conversation ID
- `lastMessageId`: Optional, ID of the last message received

**Response:**
```json
{
  "success": true,
  "message": "Latest messages retrieved successfully",
  "data": [
    {
      "id": 15,
      "body": "New message!",
      "created_at": "2024-01-01T12:05:00.000000Z",
      "sender": {
        "id": 2,
        "name": "Jane Smith"
      },
      "media": []
    }
  ]
}
```

#### 2.4 Delete Message

**Endpoint:** `DELETE /api/chats/messages/{id}`

**Description:** Deletes a specific message. **⚠️ Note: This endpoint is defined in routes but the controller method is missing.**

**Response:**
```json
{
  "success": true,
  "message": "Message deleted successfully"
}
```

### 3. Media Management

#### 3.1 Get Allowed Media Types

**Endpoint:** `GET /api/chats/media/allowed-types`

**Description:** Returns the list of allowed file types for media uploads and S3 configuration status.

**Response:**
```json
{
  "types": [
    "image/jpeg",
    "image/png",
    "image/gif",
    "video/mp4",
    "audio/mpeg"
  ],
  "s3_configured": true,
  "message": "S3 is properly configured"
}
```

#### 3.2 Get S3 Configuration Status

**Endpoint:** `GET /api/chats/media/s3-status`

**Description:** Returns S3 configuration status without exposing sensitive credentials.

**Response:**
```json
{
  "configured": true,
  "bucket": "my-bucket",
  "region": "us-east-1",
  "has_key": true,
  "has_secret": true,
  "url": "https://s3.amazonaws.com",
  "endpoint": null
}
```

#### 3.3 Test S3 Connection

**Endpoint:** `GET /api/chats/media/test-s3`

**Description:** Tests the S3 connection and configuration.

**Response:**
```json
{
  "message": "S3 connection test successful",
  "bucket": "my-bucket",
  "region": "us-east-1",
  "configured": true
}
```

### 4. Typing Indicators

#### 4.1 Start Typing Indicator

**Endpoint:** `POST /api/chats/typing/start`

**Description:** Broadcasts a typing indicator to all users in a conversation.

**Request Body:**
```json
{
  "conversation_id": 1
}
```

**Validation Rules:**
- `conversation_id`: Required, must exist in conversations table

**Features:**
- Broadcasts real-time typing events
- Excludes the current user from the broadcast
- Includes user information (name, avatar)

**Response:**
```json
{
  "success": true,
  "message": "Typing indicator started"
}
```

#### 4.2 Stop Typing Indicator

**Endpoint:** `POST /api/chats/typing/stop`

**Description:** Stops the typing indicator and notifies other users.

**Request Body:**
```json
{
  "conversation_id": 1
}
```

**Validation Rules:**
- `conversation_id`: Required, must exist in conversations table

**Response:**
```json
{
  "success": true,
  "message": "Typing indicator stopped"
}
```

### 5. Online Status Management

#### 5.1 Update Online Status

**Endpoint:** `POST /api/chats/online-status/update`

**Description:** Updates the user's online status and optionally broadcasts to a specific conversation.

**Request Body:**
```json
{
  "is_online": true,
  "conversation_id": 1
}
```

**Validation Rules:**
- `is_online`: Required boolean
- `conversation_id`: Optional, must exist in conversations table

**Features:**
- Caches online status for 5 minutes (online) or 24 hours (offline)
- Broadcasts status changes to conversation participants
- Includes user information in broadcasts

**Response:**
```json
{
  "success": true,
  "message": "Online status updated successfully"
}
```

#### 5.2 Get Conversation Online Status

**Endpoint:** `GET /api/chats/conversations/{id}/online-status`

**Description:** Retrieves online status for all users in a specific conversation.

**Response:**
```json
{
  "success": true,
  "message": "Online status retrieved successfully",
  "data": {
    "online_users": [
      {
        "user_id": 2,
        "user_name": "Jane Smith",
        "user_avatar": "https://example.com/avatar.jpg",
        "last_seen": "2024-01-01T12:00:00.000000Z"
      }
    ],
    "offline_users": [
      {
        "user_id": 3,
        "user_name": "Bob Johnson",
        "user_avatar": "https://example.com/avatar.jpg",
        "last_seen": "2024-01-01T11:30:00.000000Z"
      }
    ],
    "online_count": 1,
    "total_users": 2,
    "conversation_type": "group"
  }
}
```

#### 5.3 Get User Online Status

**Endpoint:** `GET /api/chats/users/{id}/online-status`

**Description:** Retrieves online status for a specific user (only if they share a conversation with the authenticated user).

**Response:**
```json
{
  "success": true,
  "message": "User status retrieved successfully",
  "data": {
    "user_id": 2,
    "user_name": "Jane Smith",
    "user_avatar": "https://example.com/avatar.jpg",
    "is_online": true,
    "last_seen": "2024-01-01T12:00:00.000000Z"
  }
}
```

#### 5.4 Heartbeat

**Endpoint:** `POST /api/chats/online-status/heartbeat`

**Description:** Keeps the user's online status active and optionally broadcasts to a conversation.

**Request Body:**
```json
{
  "conversation_id": 1
}
```

**Features:**
- Extends online status cache by 5 minutes
- Broadcasts heartbeat to conversation participants
- Used for maintaining real-time presence

**Response:**
```json
{
  "success": true,
  "message": "Heartbeat received"
}
```

---

## 🔄 Real-Time Events

The chat system uses Laravel Broadcasting for real-time features. The following events are broadcast:

### MessageSent Event
- **Triggered:** When a new message is sent
- **Channels:** Private channels for each user
- **Data:** Message object with sender and media information

### TypingEvent Event
- **Triggered:** When typing starts/stops
- **Channels:** Private channels for conversation participants
- **Data:** User information and typing status

### UserOnlineStatusEvent Event
- **Triggered:** When online status changes
- **Channels:** Private channels for conversation participants
- **Data:** User information and online status

---

## ⚠️ Error Handling

All endpoints return consistent error responses:

```json
{
  "success": false,
  "message": "Error description",
  "code": 400
}
```

### Common Error Codes:
- `400`: Bad Request (validation errors)
- `401`: Unauthorized (authentication required)
- `403`: Forbidden (insufficient permissions)
- `404`: Not Found (resource doesn't exist)
- `500`: Internal Server Error

---

## 🚦 Rate Limiting

The API implements rate limiting to prevent abuse. Specific limits may vary by endpoint.

---

## 📁 File Upload Guidelines

### Supported Media Types:
- **Images:** JPEG, PNG, GIF
- **Videos:** MP4, AVI, MOV
- **Audio:** MP3, WAV, AAC
- **Documents:** PDF, DOC, DOCX

### File Size Limits:
- Maximum 100MB per file
- Total upload size may be limited by server configuration

### S3 Integration:
- All media files are stored in AWS S3
- Files are processed through the S3MediaService
- URLs are generated for secure access

---

## 📝 Implementation Notes

### Missing Features:
1. **Message Deletion**: The `destroy` method in MessageController is missing but the route exists
2. **Message Editing**: No endpoint for editing existing messages
3. **Message Reactions**: No support for message reactions/emojis
4. **Message Search**: No search functionality for messages
5. **Message Forwarding**: No ability to forward messages

### Security Considerations:
- All endpoints validate user permissions
- Users can only access conversations they're part of
- Media uploads are validated for type and size
- S3 credentials are not exposed in API responses

### Performance Considerations:
- Messages are paginated (50 per page by default)
- Online status is cached to reduce database queries
- Real-time events use efficient broadcasting channels
- Media files are processed asynchronously

---

## 🎨 Frontend Integration

### Real-Time Setup:
```javascript
// Example WebSocket connection setup
const echo = new Echo({
  broadcaster: 'pusher',
  key: 'your-pusher-key',
  cluster: 'your-cluster',
  auth: {
    headers: {
      Authorization: `Bearer ${token}`
    }
  }
});

// Listen for new messages
echo.private(`user.${userId}`)
  .listen('MessageSent', (e) => {
    console.log('New message:', e.message);
  });

// Listen for typing indicators
echo.private(`conversation.${conversationId}`)
  .listen('TypingEvent', (e) => {
    console.log('User typing:', e.userName, e.isTyping);
  });
```

### Polling for Messages:
```javascript
// Poll for new messages
setInterval(() => {
  fetch(`/api/chats/conversations/poll/${conversationId}/${lastMessageId}`)
    .then(response => response.json())
    .then(data => {
      if (data.data.length > 0) {
        // Update UI with new messages
        updateMessages(data.data);
        lastMessageId = data.data[data.data.length - 1].id;
      }
    });
}, 5000); // Poll every 5 seconds
```

### Example Usage with Fetch API:
```javascript
// Send a message
const sendMessage = async (conversationId, body, media = []) => {
  const formData = new FormData();
  formData.append('conversation_id', conversationId);
  formData.append('body', body);
  
  media.forEach(file => {
    formData.append('media[]', file);
  });

  const response = await fetch('/api/chats/messages', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Accept': 'application/json',
    },
    body: formData
  });

  return response.json();
};

// Get conversations
const getConversations = async () => {
  const response = await fetch('/api/chats/conversations', {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Accept': 'application/json',
    }
  });

  return response.json();
};
```

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Setup:
```bash
# Clone the repository
git clone https://github.com/yourusername/MLS_backend.git

# Install dependencies
composer install

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Run migrations
php artisan migrate

# Start the development server
php artisan serve
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📞 Support

If you have any questions or need support, please:

- 📧 Email: support@example.com
- 🐛 Report bugs: [GitHub Issues](https://github.com/yourusername/MLS_backend/issues)
- 💬 Discuss: [GitHub Discussions](https://github.com/yourusername/MLS_backend/discussions)

---

**Made with ❤️ using Laravel** 
