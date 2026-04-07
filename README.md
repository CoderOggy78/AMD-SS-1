## What's Live in the App

6 fully functional pages with real Claude AI integration:

Dashboard — Hero banner with personalized greeting, 4 metric cards (calories, protein, water, health score), weekly calorie bar chart, macro progress bars, Recipe of the Day, water tracker with interactive drops, AI behavior insights, and today's meal log.

AI Tracker — Drag-and-drop image upload that calls Claude's vision API in real-time to identify food, estimate calories, break down protein/carbs/fats with confidence score, and log meals. Falls back to smart defaults if API is unavailable.

Recipes — 6 recipe cards with Veg/Non-Veg/Vegan + goal filters, AI-powered personalized recipe suggestions via live Claude API, full recipe detail modal with ingredients and macros.

Planner — 7-day weekly planner with day selection chips, drag-to-fill meal slots (breakfast/lunch/snack/dinner), AI week plan generation, grocery list auto-generator with checkbox state, and meal reminders panel.

Health Tools — 4 tabs: BMI calculator (live calculation + category + recommendations), Health Score with radar chart + gamified achievements, AI nutritionist chat (live Claude API, full conversation history), and multi-language support (10 languages including Hindi, Tamil, Bengali, Arabic).

Profile — User profile form, 7-week weight + health score progress chart, stats cards, allergy tracker.

## Full Production Architecture

        Folder Structure
    nutrismart-ai/
    ├── apps/
    │   ├── web/                          # Next.js 14 frontend
    │   │   ├── app/
    │   │   │   ├── (auth)/
    │   │   │   │   ├── login/page.tsx
    │   │   │   │   └── signup/page.tsx
    │   │   │   ├── dashboard/page.tsx
    │   │   │   ├── tracker/page.tsx
    │   │   │   ├── recipes/page.tsx
    │   │   │   ├── planner/page.tsx
    │   │   │   ├── tools/page.tsx
    │   │   │   ├── profile/page.tsx
    │   │   │   ├── layout.tsx
    │   │   │   └── globals.css
    │   │   ├── components/
    │   │   │   ├── ui/                   # Base components
    │   │   │   ├── charts/               # Recharts wrappers
    │   │   │   ├── tracker/              # Food upload + AI result
    │   │   │   ├── recipes/              # Recipe cards, filters
    │   │   │   └── planner/              # Meal slots, grocery
    │   │   ├── hooks/
    │   │   │   ├── useAuth.ts
    │   │   │   ├── useCalories.ts
    │   │   │   └── useAI.ts
    │   │   ├── lib/
    │   │   │   ├── firebase.ts
    │   │   │   ├── anthropic.ts
    │   │   │   └── nutrition-db.ts
    │   │   └── stores/                   # Zustand state
    │   └── mobile/                       # React Native (Phase 2)
    ├── packages/
    │   └── shared/                       # Shared types/utils
    ├── backend/
    │   ├── src/
    │   │   ├── routes/
    │   │   │   ├── auth.ts
    │   │   │   ├── meals.ts
    │   │   │   ├── recipes.ts
    │   │   │   ├── ai.ts               # Vision + chat proxy
    │   │   │   └── users.ts
    │   │   ├── middleware/
    │   │   │   ├── auth.ts             # Firebase token verify
    │   │   │   └── rateLimit.ts
    │   │   ├── services/
    │   │   │   ├── anthropicService.ts
    │   │   │   ├── nutritionix.ts      # Barcode + food DB
    │   │   │   └── scheduler.ts        # Meal reminders
    │   │   └── models/                 # Mongoose schemas
    │   └── firebase/
    │       └── functions/              # Serverless functions
    └── infrastructure/
        ├── docker-compose.yml
        └── vercel.json
        
Backend API Endpoints
        typescript// Auth
        POST   /api/auth/register
        POST   /api/auth/login
        GET    /api/auth/me

// Meals
GET    /api/meals?date=2025-04-07
POST   /api/meals                    // Manual log
DELETE /api/meals/:id

// AI
POST   /api/ai/analyze-image         // Vision food detection
POST   /api/ai/chat                  // Nutritionist chat
POST   /api/ai/recommend-recipes     // Personalized suggestions

// Recipes
GET    /api/recipes?diet=veg&goal=loss
GET    /api/recipes/:id
GET    /api/recipes/daily            // Recipe of the day (cached 24h)

// Planner
GET    /api/planner/week/:userId
PUT    /api/planner/week/:userId
GET    /api/planner/grocery-list

// Profile
GET    /api/users/:id
PUT    /api/users/:id
GET    /api/users/:id/stats
AI Integration Logic
typescript// Food Vision Analysis
async function analyzeFoodImage(imageBase64: string, mimeType: string) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 500,
    messages: [{
      role: 'user',
      content: [
        { type: 'image', source: { type: 'base64', media_type: mimeType, data: imageBase64 } },
        { type: 'text', text: `Analyze this food image. Return JSON:
          { foods: string[], totalKcal: number, protein: number,
            carbs: number, fat: number, fiber: number,
            confidence: number, portionSize: string, tip: string }` }
      ]
    }]
  });
  return JSON.parse(response.content[0].text);
}

// Personalized Recipe AI
async function getPersonalizedRecipes(userProfile: UserProfile, recentMeals: Meal[]) {
  // Analyze nutritional gaps, preferences, past behavior
  // Return ranked recipe suggestions with reasoning
}

// Behavior Analysis (runs nightly via cron)
async function generateBehaviorInsights(userId: string, weekData: MealLog[]) {
  // Meal timing patterns, nutrient deficiencies, habit loops
  // Store insights in Firestore for dashboard display
}
## Firebase Schema (Firestore)

    users/{userId}
      ├── profile: { name, age, height, weight, goal, dietPref, allergens }
      ├── settings: { calGoal, language, reminders[], notifications }
      └── stats: { streak, healthScore, level, achievements[] }
    
    meals/{mealId}
      ├── userId, date, type (breakfast/lunch/snack/dinner)
      ├── items: [{ name, kcal, protein, carbs, fat, confidence }]
      └── imageUrl, source (ai|manual|barcode), timestamp
    
    planner/{userId}/{weekId}
      └── days: { monday: { breakfast: recipeId, lunch: recipeId, ... }, ... }
    
    recipes/{recipeId}
      ├── name, emoji, kcal, macros, time, tags
      ├── diet: veg|nonveg|vegan, goal: loss|gain|maintain
      └── ingredients[], steps[], rating
    
    insights/{userId}
      └── generated daily: { behaviorTips[], mealSuggestions[], warnings[] }
    Deployment
    bash# Frontend — Vercel (zero config)
    npx vercel --prod
    
    # Backend — Railway or Render
    docker build -t nutrismart-api .
    railway up
      
    # Environment variables needed:
    ANTHROPIC_API_KEY=sk-ant-...
    FIREBASE_PROJECT_ID=nutrismart-prod
    FIREBASE_SERVICE_ACCOUNT=...
    NUTRITIONIX_API_KEY=...        # For barcode scanning
    NEXT_PUBLIC_API_URL=https://api.nutrismart.app

MVP vs Full Product Breakdown
MVP (4–6 weeks, ~$0 infra): AI image tracker + manual logging, basic dashboard with charts, BMI calculator, recipe browser, Firebase auth + Firestore storage. Deploy on Vercel free tier.
v1.0 (3 months): Meal planner + grocery list, behavior insights engine, health score gamification, water tracking, push notifications via Firebase, recipe of the day with caching.
v2.0 (6 months): Barcode scanner via Nutritionix API, React Native mobile app, multi-language support, social features (friend challenges), wearable integration (Google Fit / Apple Health).
Full Product (12 months): Custom ML model trained on user data for food detection, dietitian marketplace, telehealth integration, enterprise B2B (corporate wellness), white-label SDK.

