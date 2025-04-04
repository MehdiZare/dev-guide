# Feature Flag Management

This guide outlines our organization's standards for implementing and managing feature flags across our applications.

> 📌 **Return to**: [Main Development Guide](README.md)

## What Are Feature Flags?

Feature flags (also known as feature toggles or switches) are a technique that allows teams to modify system behavior without changing code. They enable us to:

- Deploy code to production that isn't yet ready for users
- Gradually roll out features to users
- A/B test different implementations
- Kill switches for emergency situations

Our adoption of feature flags came after a particularly painful release where we had to roll back a major feature that wasn't ready but was tangled with other critical changes. Since implementing our flag system, we've reduced deployment rollbacks by 85%.

## Feature Flag Architecture

We use a centralized feature flag service with client libraries for all our supported platforms:

```
┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │
│   Feature Flag  │ ◄── │ Admin Dashboard │
│     Service     │     │                 │
│                 │     │                 │
└────────┬────────┘     └─────────────────┘
         │
         │
         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Web Frontend   │     │   Backend API   │     │ Mobile Client   │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Key Components

1. **Feature Flag Service**: Central repository for flag definitions and states
2. **Admin Dashboard**: UI for managing flags and viewing analytics
3. **Client Libraries**: Code libraries for each platform to check flag states
4. **Analytics Integration**: Track feature flag impact

## Feature Flag Types

We use several types of flags for different purposes:

| Flag Type | Purpose | Lifetime | Example |
|-----------|---------|----------|---------|
| **Release Flags** | Control feature visibility | Temporary | `enable-new-dashboard` |
| **Experiment Flags** | A/B testing | Temporary | `pricing-model-b` |
| **Ops Flags** | System behavior control | Permanent | `high-availability-mode` |
| **Permission Flags** | Control access to features | Permanent | `admin-tools-access` |
| **Kill Switches** | Emergency feature disabling | Permanent | `disable-recommendations` |

## Naming Conventions

Feature flag names must follow these conventions:

- Use kebab-case for flag names
- Use verb-noun structure for clarity
- Include module/feature prefix for organization
- Be descriptive but concise

Examples:
```
# Good
enable-dark-mode
disable-legacy-billing
show-beta-features
use-new-search-algorithm

# Bad
darkMode
new_feature
beta
v2
```

## Implementation Patterns

### Backend Implementation (Python/FastAPI)

```python
from feature_flags import FeatureFlags

# Initialize the feature flag client
flags = FeatureFlags(api_key="your-api-key")

@app.get("/api/products")
async def get_products(user_id: str = None):
    # Check if new product recommendation algorithm is enabled
    if flags.is_enabled("use-new-recommendation-algo", user_id=user_id):
        return new_recommendation_algorithm()
    else:
        return legacy_recommendation_algorithm()
```

### Frontend Implementation (React/Next.js)

```javascript
import { useFlags } from '@feature-flags/react';

function ProductPage({ user }) {
  // Get flag state
  const { flags } = useFlags({
    userId: user.id,
    attributes: {
      userType: user.type,
      region: user.region
    }
  });
  
  return (
    <div>
      <h1>Products</h1>
      
      {flags['show-new-product-card'] && (
        <NewProductCard />
      )}
      
      <ProductList 
        useNewDesign={flags['enable-product-redesign']} 
      />
    </div>
  );
}
```

### Progressive Feature Rollout

For gradual rollouts, configure percentage-based targeting:

```python
# Server-side rollout logic
if flags.is_enabled("new-checkout-flow", 
                   user_id=user.id,
                   attributes={"user_segment": user.segment},
                   default=False):
    return new_checkout_flow()
else:
    return current_checkout_flow()
```

### Flag Dependencies

For complex features with dependencies:

```javascript
// Check parent flag first, then child flag
const showBetaFeatures = flags['enable-beta-features'];
const showNewDashboard = showBetaFeatures && flags['enable-new-dashboard'];

// Render UI based on flags
if (showNewDashboard) {
  return <NewDashboard />;
} else {
  return <CurrentDashboard />;
}
```

## Testing with Feature Flags

### Unit Testing

```python
# Python unit test with feature flags
def test_product_recommendations_with_new_algorithm():
    # Mock the feature flag to be enabled
    with mock.patch('feature_flags.is_enabled', return_value=True):
        result = get_product_recommendations(user_id="123")
        assert len(result) == 5
        # Additional assertions for new algorithm

def test_product_recommendations_with_legacy_algorithm():
    # Mock the feature flag to be disabled
    with mock.patch('feature_flags.is_enabled', return_value=False):
        result = get_product_recommendations(user_id="123")
        assert len(result) == 10
        # Additional assertions for legacy algorithm
```

### Integration Testing

```javascript
// JavaScript integration test with feature flags
describe('Product Page', () => {
  it('should show new product card when flag is enabled', async () => {
    // Mock feature flag service to enable the flag
    mockFeatureFlags({
      'show-new-product-card': true
    });
    
    render(<ProductPage user={testUser} />);
    expect(screen.getByTestId('new-product-card')).toBeInTheDocument();
  });
  
  it('should hide new product card when flag is disabled', async () => {
    // Mock feature flag service to disable the flag
    mockFeatureFlags({
      'show-new-product-card': false
    });
    
    render(<ProductPage user={testUser} />);
    expect(screen.queryByTestId('new-product-card')).not.toBeInTheDocument();
  });
});
```

### Local Development Override

For local development, provide a way to override flags:

```javascript
// Override flags for local development
if (process.env.NODE_ENV === 'development') {
  window.setFeatureFlags({
    'enable-new-dashboard': true,
    'use-new-recommendation-algo': true
  });
}
```

## Flag Lifecycle Management

### Creating a New Flag

1. Define the flag in the admin dashboard with:
    - Name and description
    - Type and default state
    - Target environments
    - Expected lifetime
    - Owner

2. Implement the flag in code
3. Deploy the code (initially with the flag off)
4. Enable the flag for testing
5. Gradually roll out to users

### Retiring a Flag

When a feature is fully rolled out and stable:

1. Make the feature permanent in code (remove conditional logic)
2. Remove flag checks from the codebase
3. Mark the flag as deprecated in the admin dashboard
4. After a deprecation period, delete the flag

### Dead Flag Detection

We've implemented automated detection of "dead flags" - flags that exist in the system but are no longer referenced in code. Our technical debt management process includes quarterly reviews of these flags.

## Feature Flag Best Practices

### Do's

- ✅ Keep flags simple and focused on a single feature
- ✅ Document all flags with clear descriptions and owners
- ✅ Clean up flags after features are fully deployed
- ✅ Use flags for all significant new features
- ✅ Test both flag states (on and off)

### Don'ts

- ❌ Create flags that control multiple unrelated features
- ❌ Nest flag conditions too deeply (avoid complex flag trees)
- ❌ Leave temporary flags in the codebase indefinitely
- ❌ Use feature flags for configuration values
- ❌ Assume a flag state without checking

## Analytics and Monitoring

### Tracking Flag Impact

Connect feature flags to business metrics:

```javascript
// Track feature flag impact on conversion
function handlePurchase(purchaseData) {
  analytics.track('Purchase Completed', {
    ...purchaseData,
    activeFeatureFlags: {
      'new-checkout-flow': flags['new-checkout-flow'],
      'pricing-experiment': flags['pricing-experiment']
    }
  });
}
```

### Flag Status Monitoring

Monitor feature flag status changes:

```python
# Log flag status changes
logging.info(
    "Feature flag status change", 
    extra={
        "flag_name": "new-checkout-flow",
        "flag_state": True,
        "environment": "production"
    }
)
```

## Cross-Functional Coordination

For successful feature flag management:

1. **Product Managers**: Define feature requirements and rollout plans
2. **Developers**: Implement flags in code and ensure proper testing
3. **QA**: Test both flag states before deployment
4. **Operations**: Monitor flag impact on system performance
5. **Support**: Stay informed about feature releases to assist users

Our flag review process involves weekly meetings with representatives from each team to coordinate flag states and rollout plans.

## Security Considerations

Treat feature flags as a potential security boundary:

1. Implement proper access controls to flag management
2. Audit all flag changes
3. Use caution with flags that modify authentication or authorization
4. Consider encryption for sensitive flags

## Case Studies

### Gradual Rollout: New Checkout Flow

When rebuilding our checkout flow, we used feature flags to gradually roll out the changes:

1. Created `enable-new-checkout` flag
2. Deployed code with flag disabled
3. Enabled for internal users (10%)
4. Gradually increased to 25%, 50%, 75%
5. Monitored conversion rates and error logs
6. Rolled back for a specific user segment that showed issues
7. Fixed issues and re-enabled for all users
8. Removed flag after 2 weeks of stability

The gradual rollout caught a critical payment processing issue that only affected users in certain regions, preventing a potentially costly service disruption.

### A/B Testing: Pricing Display

To test different pricing display strategies:

1. Created `pricing-display-variant` flag with options "A" (original) and "B" (new)
2. Assigned users randomly to variants
3. Tracked conversion metrics for each variant
4. After 4 weeks, variant B showed 12% higher conversion
5. Rolled out variant B to all users
6. Removed the flag

### Emergency Kill Switch: Search Service

During a search service disruption:

1. Activated `disable-advanced-search` kill switch
2. Reverted to basic search functionality
3. Added monitoring to track basic search performance
4. Fixed advanced search backend issues
5. Gradually re-enabled advanced search
6. Kept the kill switch for future use

The kill switch allowed us to keep the platform operational during a critical outage, maintaining 95% of core functionality while we resolved the backend issues.

## Lessons Learned

### Avoiding Flag Debt

In our early implementation, we created many flags without a removal plan, leading to a confusing codebase with deprecated features still gated behind active flags. Now we:

1. Track flag creation date and expected retirement date
2. Assign explicit ownership to each flag
3. Include flag removal in sprint planning
4. Review flag status monthly

### Performance Considerations

Feature flag checks can impact performance if not optimized. After discovering latency issues, we:

1. Implemented client-side caching of flag states
2. Reduced flag API calls by batching requests
3. Established fallback flag states for offline operation
4. Moved critical flag checks to app initialization

### Team Communication

Poor communication about flag states led to confusion during our first major flagged release. We now:

1. Maintain a shared feature flag dashboard
2. Send notifications for flag state changes
3. Include flag status in release notes
4. Document flag dependencies explicitly

## Tools and Services

We currently use the following for feature flag management:

1. **Self-hosted solution**: Our internal feature flag service
2. **Client libraries**: Custom SDKs for each platform
3. **Monitoring**: Integration with our observability platform
4. **Documentation**: Wiki page for each flag with status and ownership

## Migration Guide

For teams migrating to our feature flag system:

1. Identify existing feature toggle mechanisms
2. Create equivalent flags in the centralized system
3. Replace custom toggle code with feature flag SDK calls
4. Validate behavior in all environments
5. Remove legacy toggle mechanisms