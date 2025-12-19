# Real-World System Examples

This document provides real-world examples of systems you use every day and how they're designed.

## Example 1: Instagram

### System Overview
Instagram is a photo and video sharing social network with over 2 billion users.

### Key Components

**Frontend:**
- Mobile apps (iOS, Android)
- Web application
- Stories feature
- Reels feature

**Backend Services:**
- User authentication service
- Photo upload service
- Feed generation service
- Search and discovery service
- Messaging service
- Notification service

**Data Storage:**
- User profiles database
- Photos and videos (object storage)
- Relationships (followers/following)
- Comments and likes
- Direct messages

**Infrastructure:**
- CDN for fast image delivery worldwide
- Caching for frequently accessed feeds
- Load balancers distributing traffic
- Message queues for asynchronous tasks

### Scale Numbers
- 2 billion+ users
- 95 million posts per day
- 4.2 billion likes per day
- Serving content in < 200ms globally

### Design Decisions

**Why Use a CDN?**
- Photos need to load fast worldwide
- Storing copies closer to users reduces latency
- Reduces load on origin servers

**Why Use Caching?**
- Your feed doesn't change every second
- Cache feed for a few minutes
- Dramatically reduces database queries

**Why Use Message Queues?**
- Notifications don't need to be instant
- Process them asynchronously
- Handle millions without slowing down uploads

---

## Example 2: Netflix

### System Overview
Netflix is a video streaming platform serving 230+ million subscribers worldwide.

### Key Components

**Frontend:**
- Smart TV apps
- Mobile apps (iOS, Android)
- Web browsers
- Game consoles

**Backend Services:**
- User authentication
- Content recommendation engine
- Video streaming service
- Playback quality adjustment
- Subtitle/audio track service
- Billing service

**Data Storage:**
- User profiles and preferences
- Watch history and ratings
- Video files in multiple qualities
- Metadata (titles, descriptions, thumbnails)

**Infrastructure:**
- Massive CDN (Netflix Open Connect)
- Encoding service (creates multiple quality versions)
- Recommendation ML models
- A/B testing platform

### Scale Numbers
- 230 million+ subscribers
- 1 billion+ hours watched weekly
- Petabytes of video data
- Content in 190+ countries

### Design Decisions

**Why Multiple Video Qualities?**
- Users have different internet speeds
- Automatically adjust quality based on bandwidth
- Prevents buffering

**Why Pre-encode Videos?**
- Don't encode in real-time (too slow)
- Create multiple versions ahead of time
- Different resolutions: 4K, 1080p, 720p, 480p

**Why Own CDN Infrastructure?**
- Video streaming is bandwidth-intensive
- Cheaper to build own CDN at Netflix's scale
- Better control over quality

---

## Example 3: WhatsApp

### System Overview
WhatsApp is a messaging app with end-to-end encryption, serving 2+ billion users.

### Key Components

**Frontend:**
- Mobile apps (iOS, Android)
- Desktop application
- Web application

**Backend Services:**
- Message routing service
- Media upload/download service
- Group management service
- Status/Stories service
- Call routing (voice and video)

**Data Storage:**
- User profiles (phone numbers, names)
- Contact lists
- Media files (photos, videos, voice notes)
- Message metadata (not content - it's encrypted!)

**Infrastructure:**
- WebSocket connections (real-time)
- Media CDN for photos/videos
- Load balancers
- End-to-end encryption

### Scale Numbers
- 2 billion+ users
- 100 billion+ messages per day
- 2 billion+ minutes of calls per day
- Engineered for reliability: 99.99% uptime

### Design Decisions

**Why WebSockets Instead of HTTP?**
- Messages need to be instant
- HTTP requires constant polling (inefficient)
- WebSockets maintain open connection

**Why End-to-End Encryption?**
- Privacy and security
- Even WhatsApp can't read messages
- Messages encrypted on sender's device, decrypted on receiver's

**Why Minimal Servers?**
- WhatsApp runs on surprisingly few servers
- Messages route directly between users (peer-to-peer-like)
- Servers mainly handle coordination, not storage

---

## Example 4: Uber

### System Overview
Uber connects riders with drivers for transportation services.

### Key Components

**Frontend:**
- Rider mobile app
- Driver mobile app
- Web interface for account management

**Backend Services:**
- Location service (GPS tracking)
- Matching service (pairs riders with drivers)
- Pricing service (surge pricing algorithm)
- Payment processing service
- Notification service
- Trip history and analytics
- Fraud detection service

**Data Storage:**
- User accounts (riders and drivers)
- Real-time location data
- Trip history
- Pricing data
- Payment information

**Infrastructure:**
- Real-time location tracking
- Map services (Google Maps, custom mapping)
- Payment gateway integration
- Push notification service

### Scale Numbers
- 131 million+ users
- 23 million+ trips per day
- Real-time tracking of millions of vehicles
- Operating in 70+ countries

### Design Decisions

**Why Real-Time Location Updates?**
- Riders need to see driver approaching
- Drivers need navigation
- Safety and transparency

**Why Surge Pricing?**
- Balance supply (drivers) and demand (riders)
- Incentivize more drivers during busy times
- Algorithmic pricing based on real-time data

**Why Separate Rider and Driver Apps?**
- Different workflows and features
- Optimized for specific use cases
- Easier to maintain and update

---

## Example 5: Twitter (X)

### System Overview
Twitter is a microblogging platform for sharing short messages (tweets).

### Key Components

**Frontend:**
- Mobile apps
- Web application
- Third-party apps (via API)

**Backend Services:**
- Tweet posting service
- Timeline generation service
- Search service
- Trending topics service
- Notification service
- Direct messaging service

**Data Storage:**
- User profiles
- Tweets (text, media)
- Social graph (followers/following)
- Likes, retweets, replies
- Trending data

**Infrastructure:**
- Caching (heavy caching for timelines)
- Search indexing (Elasticsearch)
- CDN for media content
- Message queues

### Scale Numbers
- 500 million+ tweets per day
- Peaks of 143,000 tweets per second
- Millions of concurrent users
- Real-time global trending topics

### Design Decisions

**Why Cache Timelines?**
- Timelines don't change constantly
- Pre-generate timelines for popular users
- Reduces database load significantly

**Why Fan-Out on Write?**
- When you tweet, system pushes to your followers' timelines
- Makes reading timelines fast
- Trade-off: Slower tweet posting for faster timeline reads

**Why Separate Search Infrastructure?**
- Search is complex (keywords, hashtags, dates)
- Dedicated search system (Elasticsearch)
- Doesn't slow down main database

---

## Example 6: Amazon

### System Overview
Amazon is a massive e-commerce platform with diverse services.

### Key Components

**Frontend:**
- Website
- Mobile apps
- Alexa voice interface
- Fire TV apps

**Backend Services:**
- Product catalog service
- Shopping cart service
- Order management service
- Payment processing service
- Inventory management service
- Recommendation engine
- Review and rating service
- Shipping and tracking service

**Data Storage:**
- Product database
- User accounts and preferences
- Order history
- Inventory levels
- Reviews and ratings
- Shopping cart data

**Infrastructure:**
- Microservices architecture (thousands of services)
- API Gateway
- Load balancers
- CDN for images and static content
- Caching at multiple levels

### Scale Numbers
- 300 million+ active customers
- 12 million+ products
- Peak: 37,000 orders per minute (Prime Day)
- 200 million+ Prime members

### Design Decisions

**Why Microservices?**
- Different teams own different features
- Services scale independently
- Shopping cart needs different scaling than payment

**Why Separate Recommendation Service?**
- Complex ML algorithms
- Can be slow without affecting other features
- Personalized for each user

**Why Multiple Caching Layers?**
- Product pages cached at CDN (static content)
- Search results cached in application layer
- Database query caching
- Reduces response time from seconds to milliseconds

---

## Example 7: YouTube

### System Overview
YouTube is a video sharing platform with 2.7+ billion users.

### Key Components

**Frontend:**
- Website
- Mobile apps
- TV apps
- Embedded players

**Backend Services:**
- Video upload and processing service
- Video streaming service
- Recommendation engine
- Comment service
- Live streaming service
- Ad serving system
- Analytics service

**Data Storage:**
- Video files (multiple qualities)
- User accounts
- Video metadata
- Comments
- View counts and analytics
- Subscriptions

**Infrastructure:**
- Global CDN
- Video transcoding (encoding to different formats)
- Adaptive bitrate streaming
- Content recommendation ML
- Copyright detection system

### Scale Numbers
- 500+ hours of video uploaded every minute
- 1 billion+ hours watched daily
- Petabytes of data stored
- Videos in 100+ languages

### Design Decisions

**Why Transcode Videos?**
- Uploaded in various formats and qualities
- Convert to standard formats
- Create multiple quality versions
- Users can watch based on their internet speed

**Why Adaptive Streaming?**
- Automatically adjusts quality during playback
- Prevents buffering
- Optimizes for user's current bandwidth

**Why Recommendation System?**
- Keeps users engaged
- 70% of watch time from recommendations
- ML models trained on viewing patterns

---

## Example 8: Spotify

### System Overview
Spotify is a music streaming service with 500+ million users.

### Key Components

**Frontend:**
- Mobile apps
- Desktop application
- Web player
- Car integration
- Smart speaker integration

**Backend Services:**
- Audio streaming service
- Playlist management service
- Recommendation engine
- Social features service
- Podcast service
- Artist analytics service

**Data Storage:**
- Music files (multiple qualities)
- User profiles and preferences
- Playlists
- Listen history
- Podcast episodes
- User-generated content

**Infrastructure:**
- Audio CDN
- Caching of popular tracks
- Offline download support
- Real-time collaboration (shared playlists)

### Scale Numbers
- 500 million+ users
- 100 million+ songs
- 5 million+ podcasts
- Billions of streams per day

### Design Decisions

**Why Cache Popular Songs?**
- Top 10% of songs account for 90% of streams
- Cache popular tracks near users
- Reduces streaming costs

**Why Different Audio Qualities?**
- Free users: Standard quality
- Premium users: High quality
- Saves bandwidth for free users

**Why Allow Offline Downloads?**
- Users can listen without internet
- Reduces streaming costs
- Better user experience

---

## Common Patterns Across Systems

### 1. Caching Everywhere
All large systems use multiple layers of caching to improve performance.

### 2. Asynchronous Processing
Long-running tasks (email, notifications) processed in background.

### 3. Microservices for Scale
Large systems break into smaller, independent services.

### 4. CDN for Global Reach
Static content served from locations close to users.

### 5. Database Replication
Read-heavy systems use multiple database copies.

### 6. Load Balancing
Traffic distributed across many servers for reliability.

### 7. Monitoring and Observability
All systems heavily monitor metrics and logs.

### 8. Gradual Feature Rollout
New features tested on small user groups first (A/B testing).

---

## Summary

Real-world systems share common patterns:
- **Start simple**, scale as needed
- **Cache aggressively** to reduce load
- **Process asynchronously** when possible
- **Use CDNs** for global performance
- **Replicate data** for reliability
- **Monitor everything** to catch issues early

These examples show that system design principles apply consistently across different types of applications, from social media to e-commerce to streaming services.

## Next Steps

After studying these examples, complete the [Design Exercises](./design-exercise.md) to practice applying these concepts!
