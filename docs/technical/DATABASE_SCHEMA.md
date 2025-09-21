# AI GALAXY - Database Schema & API Specifications

## 1. Database Architecture Overview

### 1.1 Technology Stack
- **Primary Database**: Supabase (PostgreSQL)
- **Real-time Features**: Supabase Realtime
- **Authentication**: Supabase Auth
- **Storage**: Supabase Storage for assets
- **CDN**: Supabase CDN for global content delivery

### 1.2 Schema Design Principles
- **Scalability**: Designed for millions of users and thousands of AI tools
- **Performance**: Optimized queries with proper indexing
- **Flexibility**: Easy to add new categories and features
- **Security**: Row Level Security (RLS) for data protection
- **Real-time**: Live updates for new tools and news

---

## 2. Core Database Tables

### 2.1 AI Tools Management

```sql
-- AI Tools Categories Enum
CREATE TYPE ai_category AS ENUM (
  'chat',
  'image_generation', 
  'coding',
  'music_audio',
  'video_animation',
  'writing_content',
  'productivity',
  'design_creative',
  'data_analytics',
  'education_learning',
  'business',
  'research',
  'gaming',
  'social'
);

-- Main AI Tools Table
CREATE TABLE ai_tools (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT NOT NULL,
  short_description TEXT,
  category ai_category NOT NULL,
  subcategory TEXT,
  
  -- URLs and Links
  website_url TEXT NOT NULL,
  logo_url TEXT,
  banner_url TEXT,
  
  -- Metrics and Popularity
  popularity_score INTEGER DEFAULT 0,
  rating DECIMAL(3,2) DEFAULT 0.0,
  review_count INTEGER DEFAULT 0,
  monthly_users INTEGER DEFAULT 0,
  
  -- Features and Pricing
  is_free BOOLEAN DEFAULT true,
  has_paid_plan BOOLEAN DEFAULT false,
  pricing_model TEXT, -- 'free', 'freemium', 'paid', 'subscription'
  starting_price DECIMAL(10,2),
  price_currency TEXT DEFAULT 'USD',
  
  -- Status and Visibility
  is_featured BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  is_verified BOOLEAN DEFAULT false,
  launch_date DATE,
  
  -- 3D Visualization Properties
  planet_size DECIMAL(3,2) DEFAULT 1.0, -- Visual size multiplier
  planet_color TEXT DEFAULT '#0099FF',
  orbit_radius INTEGER DEFAULT 400,
  orbit_speed DECIMAL(5,3) DEFAULT 1.0,
  
  -- SEO and Discovery
  tags TEXT[], -- Array of searchable tags
  keywords TEXT[], -- SEO keywords
  slug TEXT UNIQUE, -- URL-friendly identifier
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  last_verified_at TIMESTAMPTZ,
  
  -- Constraints
  CONSTRAINT valid_rating CHECK (rating >= 0 AND rating <= 5),
  CONSTRAINT valid_planet_size CHECK (planet_size >= 0.3 AND planet_size <= 2.0),
  CONSTRAINT valid_orbit_radius CHECK (orbit_radius >= 100 AND orbit_radius <= 1000)
);

-- Indexes for performance
CREATE INDEX idx_ai_tools_category ON ai_tools(category);
CREATE INDEX idx_ai_tools_popularity ON ai_tools(popularity_score DESC);
CREATE INDEX idx_ai_tools_featured ON ai_tools(is_featured) WHERE is_featured = true;
CREATE INDEX idx_ai_tools_active ON ai_tools(is_active) WHERE is_active = true;
CREATE INDEX idx_ai_tools_search ON ai_tools USING GIN(to_tsvector('english', name || ' ' || description || ' ' || array_to_string(tags, ' ')));
CREATE INDEX idx_ai_tools_slug ON ai_tools(slug);
```

### 2.2 User Management and Preferences

```sql
-- User Profiles (extends Supabase auth.users)
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  username TEXT UNIQUE,
  display_name TEXT,
  avatar_url TEXT,
  bio TEXT,
  
  -- Preferences
  theme_preference TEXT DEFAULT 'dark', -- 'dark', 'light', 'auto'
  language_preference TEXT DEFAULT 'en',
  notification_preferences JSONB DEFAULT '{"new_tools": true, "updates": true, "weekly_digest": true}',
  
  -- Galaxy Customization
  preferred_galaxy_view TEXT DEFAULT 'category', -- 'category', 'popularity', 'recent'
  custom_galaxy_settings JSONB DEFAULT '{}',
  
  -- Privacy Settings
  profile_visibility TEXT DEFAULT 'public', -- 'public', 'private'
  show_activity BOOLEAN DEFAULT true,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  last_seen_at TIMESTAMPTZ DEFAULT now()
);

-- User Favorites/Bookmarks
CREATE TABLE user_favorites (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  tool_id UUID NOT NULL REFERENCES ai_tools(id) ON DELETE CASCADE,
  
  -- Organization
  folder_name TEXT DEFAULT 'My Favorites',
  notes TEXT, -- Personal notes about the tool
  
  -- Usage Tracking
  times_opened INTEGER DEFAULT 0,
  last_opened_at TIMESTAMPTZ,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  
  -- Constraints
  UNIQUE(user_id, tool_id)
);

-- User Activity Tracking
CREATE TABLE user_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Activity Details
  activity_type TEXT NOT NULL, -- 'tool_view', 'tool_click', 'search', 'favorite_add', etc.
  tool_id UUID REFERENCES ai_tools(id) ON DELETE SET NULL,
  metadata JSONB DEFAULT '{}', -- Additional activity data
  
  -- Context
  session_id UUID,
  ip_address INET,
  user_agent TEXT,
  
  -- Timestamp
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Indexes
CREATE INDEX idx_user_favorites_user ON user_favorites(user_id);
CREATE INDEX idx_user_activities_user_type ON user_activities(user_id, activity_type);
CREATE INDEX idx_user_activities_tool ON user_activities(tool_id) WHERE tool_id IS NOT NULL;
```

### 2.3 Content and News Management

```sql
-- News and Updates
CREATE TABLE ai_news (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- Content
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  excerpt TEXT,
  image_url TEXT,
  
  -- Attribution
  author_name TEXT,
  author_avatar TEXT,
  source_name TEXT,
  source_url TEXT,
  
  -- Categorization
  category TEXT DEFAULT 'general', -- 'general', 'product_update', 'industry_news', 'tutorial'
  tags TEXT[],
  related_tools UUID[], -- Array of AI tool IDs
  
  -- Publishing
  status TEXT DEFAULT 'draft', -- 'draft', 'published', 'archived'
  published_at TIMESTAMPTZ,
  featured_until TIMESTAMPTZ,
  
  -- Engagement
  view_count INTEGER DEFAULT 0,
  like_count INTEGER DEFAULT 0,
  share_count INTEGER DEFAULT 0,
  
  -- SEO
  slug TEXT UNIQUE,
  meta_description TEXT,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Tool Updates and Changelog
CREATE TABLE tool_updates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tool_id UUID NOT NULL REFERENCES ai_tools(id) ON DELETE CASCADE,
  
  -- Update Details
  version TEXT,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  update_type TEXT DEFAULT 'feature', -- 'feature', 'bugfix', 'security', 'breaking'
  
  -- Visibility
  is_major BOOLEAN DEFAULT false,
  is_public BOOLEAN DEFAULT true,
  
  -- Timestamps
  released_at TIMESTAMPTZ DEFAULT now(),
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Indexes
CREATE INDEX idx_ai_news_published ON ai_news(published_at DESC) WHERE status = 'published';
CREATE INDEX idx_ai_news_category ON ai_news(category);
CREATE INDEX idx_tool_updates_tool ON tool_updates(tool_id, released_at DESC);
```

### 2.4 Social Features and Community

```sql
-- User Reviews and Ratings
CREATE TABLE tool_reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tool_id UUID NOT NULL REFERENCES ai_tools(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Review Content
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  title TEXT,
  content TEXT,
  pros TEXT[],
  cons TEXT[],
  
  -- Context
  use_case TEXT, -- How they use the tool
  experience_level TEXT, -- 'beginner', 'intermediate', 'expert'
  
  -- Moderation
  is_verified BOOLEAN DEFAULT false,
  is_hidden BOOLEAN DEFAULT false,
  moderation_notes TEXT,
  
  -- Engagement
  helpful_count INTEGER DEFAULT 0,
  report_count INTEGER DEFAULT 0,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  
  -- Constraints
  UNIQUE(tool_id, user_id) -- One review per user per tool
);

-- User Collections (Custom Galaxy Views)
CREATE TABLE user_collections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Collection Details
  name TEXT NOT NULL,
  description TEXT,
  cover_image TEXT,
  
  -- Settings
  is_public BOOLEAN DEFAULT false,
  is_featured BOOLEAN DEFAULT false,
  
  -- Galaxy Visualization
  custom_layout JSONB DEFAULT '{}', -- Custom positioning and styling
  
  -- Engagement
  view_count INTEGER DEFAULT 0,
  like_count INTEGER DEFAULT 0,
  fork_count INTEGER DEFAULT 0,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Collection Items (Tools in Collections)
CREATE TABLE collection_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  collection_id UUID NOT NULL REFERENCES user_collections(id) ON DELETE CASCADE,
  tool_id UUID NOT NULL REFERENCES ai_tools(id) ON DELETE CASCADE,
  
  -- Organization
  position INTEGER,
  notes TEXT,
  
  -- Custom Styling (for personalized galaxy view)
  custom_position JSONB, -- {x, y, z} coordinates
  custom_size DECIMAL(3,2),
  custom_color TEXT,
  
  -- Timestamps
  added_at TIMESTAMPTZ DEFAULT now(),
  
  -- Constraints
  UNIQUE(collection_id, tool_id)
);
```

---

## 3. Row Level Security (RLS) Policies

```sql
-- Enable RLS on all tables
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_favorites ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_activities ENABLE ROW LEVEL SECURITY;
ALTER TABLE tool_reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_collections ENABLE ROW LEVEL SECURITY;
ALTER TABLE collection_items ENABLE ROW LEVEL SECURITY;

-- User Profile Policies
CREATE POLICY "Users can view public profiles"
  ON user_profiles FOR SELECT
  TO authenticated
  USING (profile_visibility = 'public' OR auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON user_profiles FOR UPDATE
  TO authenticated
  USING (auth.uid() = id);

-- User Favorites Policies
CREATE POLICY "Users can manage own favorites"
  ON user_favorites FOR ALL
  TO authenticated
  USING (auth.uid() = user_id);

-- Reviews Policies
CREATE POLICY "Anyone can view non-hidden reviews"
  ON tool_reviews FOR SELECT
  TO authenticated
  USING (NOT is_hidden);

CREATE POLICY "Users can manage own reviews"
  ON tool_reviews FOR ALL
  TO authenticated
  USING (auth.uid() = user_id);

-- Collections Policies
CREATE POLICY "Anyone can view public collections"
  ON user_collections FOR SELECT
  TO authenticated
  USING (is_public = true OR auth.uid() = user_id);

CREATE POLICY "Users can manage own collections"
  ON user_collections FOR ALL
  TO authenticated
  USING (auth.uid() = user_id);
```

---

## 4. API Endpoints Specification

### 4.1 AI Tools API

```typescript
// GET /api/tools - List all AI tools with filtering
interface GetToolsRequest {
  category?: AICategory;
  search?: string;
  tags?: string[];
  featured?: boolean;
  limit?: number;
  offset?: number;
  sort?: 'popularity' | 'rating' | 'recent' | 'alphabetical';
}

interface GetToolsResponse {
  tools: AITool[];
  total: number;
  pagination: {
    limit: number;
    offset: number;
    hasMore: boolean;
  };
  filters: {
    categories: { category: AICategory; count: number }[];
    tags: { tag: string; count: number }[];
  };
}

// GET /api/tools/:id - Get tool details
interface GetToolResponse extends AITool {
  reviews: {
    average: number;
    count: number;
    distribution: { [rating: number]: number };
    recent: ToolReview[];
  };
  updates: ToolUpdate[];
  relatedTools: AITool[];
  userInteraction?: {
    isFavorited: boolean;
    hasReviewed: boolean;
    timesOpened: number;
  };
}

// POST /api/tools/:id/favorite - Toggle favorite
interface ToggleFavoriteRequest {
  folder?: string;
  notes?: string;
}

// POST /api/tools/:id/open - Track tool access
interface TrackToolOpenRequest {
  source: 'galaxy' | 'search' | 'favorites' | 'news';
}
```

### 4.2 Search and Discovery API

```typescript
// GET /api/search - Universal search
interface SearchRequest {
  q: string;
  types?: ('tools' | 'news' | 'collections')[];
  filters?: {
    category?: AICategory[];
    priceRange?: 'free' | 'freemium' | 'paid';
    rating?: number; // Minimum rating
  };
  limit?: number;
  offset?: number;
}

interface SearchResponse {
  results: {
    tools: AITool[];
    news: NewsArticle[];
    collections: UserCollection[];
  };
  suggestions: string[];
  facets: {
    categories: { category: AICategory; count: number }[];
    priceModels: { model: string; count: number }[];
    ratings: { range: string; count: number }[];
  };
}

// GET /api/recommendations/:userId - Personalized recommendations
interface RecommendationsResponse {
  forYou: AITool[];
  trending: AITool[];
  newReleases: AITool[];
  similarToFavorites: AITool[];
  reasoning: {
    [toolId: string]: string; // Explanation for recommendation
  };
}
```

### 4.3 User Data API

```typescript
// GET /api/user/profile - Get user profile
// PUT /api/user/profile - Update user profile
interface UserProfile {
  id: string;
  username: string;
  displayName: string;
  avatarUrl?: string;
  bio?: string;
  preferences: UserPreferences;
  stats: {
    favoritesCount: number;
    reviewsCount: number;
    collectionsCount: number;
    joinedDate: string;
  };
}

// GET /api/user/favorites - Get user favorites
interface UserFavoritesResponse {
  folders: {
    name: string;
    count: number;
    tools: AITool[];
  }[];
  total: number;
  recentlyAdded: AITool[];
}

// GET /api/user/activity - Get user activity feed
interface UserActivityResponse {
  activities: UserActivity[];
  stats: {
    toolsViewed: number;
    toolsUsed: number;
    reviewsWritten: number;
  };
}
```

### 4.4 Content API

```typescript
// GET /api/news - Get AI news and updates
interface GetNewsRequest {
  category?: 'general' | 'product_update' | 'industry_news' | 'tutorial';
  featured?: boolean;
  relatedTool?: string;
  limit?: number;
  offset?: number;
}

interface GetNewsResponse {
  articles: NewsArticle[];
  featured: NewsArticle[];
  trending: NewsArticle[];
  total: number;
}

// GET /api/updates/:toolId - Get tool update history
interface ToolUpdatesResponse {
  updates: ToolUpdate[];
  summary: {
    majorUpdates: number;
    lastUpdate: string;
    updateFrequency: 'high' | 'medium' | 'low';
  };
}
```

---

## 5. Real-time Features

### 5.1 Supabase Realtime Subscriptions

```typescript
// Real-time tool updates
const subscribeToToolUpdates = (callback: (update: ToolUpdate) => void) => {
  return supabase
    .channel('tool-updates')
    .on('postgres_changes', {
      event: '*',
      schema: 'public',
      table: 'ai_tools'
    }, callback)
    .subscribe();
};

// Real-time news feed
const subscribeToNews = (callback: (news: NewsArticle) => void) => {
  return supabase
    .channel('ai-news')
    .on('postgres_changes', {
      event: 'INSERT',
      schema: 'public',
      table: 'ai_news',
      filter: 'status=eq.published'
    }, callback)
    .subscribe();
};

// User activity notifications
const subscribeToUserNotifications = (userId: string, callback: Function) => {
  return supabase
    .channel(`user-${userId}`)
    .on('postgres_changes', {
      event: 'INSERT',
      schema: 'public',
      table: 'user_activities',
      filter: `user_id=eq.${userId}`
    }, callback)
    .subscribe();
};
```

---

## 6. Performance Optimization

### 6.1 Database Optimization

```sql
-- Materialized view for popular tools
CREATE MATERIALIZED VIEW popular_tools AS
SELECT 
  t.*,
  COALESCE(AVG(r.rating), 0) as avg_rating,
  COUNT(r.id) as review_count,
  COUNT(f.id) as favorite_count
FROM ai_tools t
LEFT JOIN tool_reviews r ON t.id = r.tool_id AND NOT r.is_hidden
LEFT JOIN user_favorites f ON t.id = f.tool_id
WHERE t.is_active = true
GROUP BY t.id
ORDER BY 
  (t.popularity_score * 0.4 + 
   COALESCE(AVG(r.rating), 0) * 20 * 0.3 + 
   COUNT(f.id) * 0.3) DESC;

-- Refresh strategy
CREATE OR REPLACE FUNCTION refresh_popular_tools()
RETURNS void AS $$
BEGIN
  REFRESH MATERIALIZED VIEW CONCURRENTLY popular_tools;
END;
$$ LANGUAGE plpgsql;

-- Auto-refresh every hour
SELECT cron.schedule('refresh-popular-tools', '0 * * * *', 'SELECT refresh_popular_tools();');
```

### 6.2 Caching Strategy

```typescript
// Redis caching for frequent queries
const CACHE_KEYS = {
  POPULAR_TOOLS: 'popular:tools',
  TOOL_DETAILS: (id: string) => `tool:${id}`,
  USER_FAVORITES: (userId: string) => `user:${userId}:favorites`,
  SEARCH_RESULTS: (query: string) => `search:${Buffer.from(query).toString('base64')}`,
  NEWS_FEED: 'news:latest'
};

const CACHE_TTL = {
  POPULAR_TOOLS: 3600,    // 1 hour
  TOOL_DETAILS: 1800,     // 30 minutes  
  USER_FAVORITES: 300,    // 5 minutes
  SEARCH_RESULTS: 900,    // 15 minutes
  NEWS_FEED: 1800         // 30 minutes
};
```

---

## 7. Data Migration and Seeding

### 7.1 Initial Data Seeding

```sql
-- Insert popular AI tools
INSERT INTO ai_tools (name, description, category, website_url, popularity_score, is_featured) VALUES
('ChatGPT', 'Advanced AI chatbot for conversation and assistance', 'chat', 'https://chat.openai.com', 100, true),
('Midjourney', 'AI-powered image generation tool', 'image_generation', 'https://midjourney.com', 95, true),
('GitHub Copilot', 'AI programming assistant', 'coding', 'https://github.com/features/copilot', 90, true),
('Claude', 'Constitutional AI assistant by Anthropic', 'chat', 'https://claude.ai', 85, true),
('DALL-E 3', 'Advanced AI image generator by OpenAI', 'image_generation', 'https://openai.com/dall-e-3', 88, true);

-- Set up initial galaxy layout
UPDATE ai_tools SET 
  orbit_radius = CASE 
    WHEN popularity_score >= 90 THEN 200  -- Inner ring for top tools
    WHEN popularity_score >= 70 THEN 350  -- Middle ring
    ELSE 500                              -- Outer ring
  END,
  orbit_speed = CASE
    WHEN popularity_score >= 90 THEN 0.8
    WHEN popularity_score >= 70 THEN 0.6
    ELSE 0.4
  END;
```

### 7.2 Migration Scripts

```typescript
// Migration utility for adding new features
export const migrationScripts = {
  addToolTags: `
    ALTER TABLE ai_tools ADD COLUMN IF NOT EXISTS tags TEXT[];
    UPDATE ai_tools SET tags = ARRAY[]::TEXT[] WHERE tags IS NULL;
  `,
  
  addUserPreferences: `
    ALTER TABLE user_profiles 
    ADD COLUMN IF NOT EXISTS notification_preferences JSONB DEFAULT '{}',
    ADD COLUMN IF NOT EXISTS custom_galaxy_settings JSONB DEFAULT '{}';
  `,
  
  createIndexes: `
    CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_ai_tools_tags 
    ON ai_tools USING GIN(tags);
    
    CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_tool_reviews_rating 
    ON tool_reviews(tool_id, rating) WHERE NOT is_hidden;
  `
};
```

This comprehensive database schema provides a solid foundation for AI GALAXY, supporting all core features while maintaining performance, security, and scalability requirements.