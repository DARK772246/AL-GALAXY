# AI GALAXY - UI Component Library & Glassmorphism Styling

## 1. Design System Foundation

### 1.1 Design Tokens

```typescript
// Theme Configuration
export const GALAXY_THEME = {
  colors: {
    // Primary Palette
    cosmic: {
      black: '#0a0a0a',
      deepSpace: '#1a1a2e',
      nebula: '#16213e',
      stellar: '#0f3460',
      void: '#000000'
    },
    
    // Accent Colors
    neon: {
      cyan: '#00d4ff',
      pink: '#ff006e',
      green: '#39ff14',
      orange: '#ff4500',
      gold: '#ffb700',
      purple: '#a855f7'
    },
    
    // UI Colors
    ui: {
      white: '#ffffff',
      gray100: '#f7f7f7',
      gray200: '#e5e5e5',
      gray300: '#d1d5db',
      gray400: '#9ca3af',
      gray500: '#6b7280',
      gray600: '#4b5563',
      gray700: '#374151',
      gray800: '#1f2937',
      gray900: '#111827'
    },
    
    // Semantic Colors
    semantic: {
      success: '#10b981',
      warning: '#f59e0b',
      error: '#ef4444',
      info: '#3b82f6'
    }
  },
  
  // Typography Scale
  typography: {
    fontFamily: {
      primary: 'Orbitron', // Futuristic headers
      secondary: 'Exo 2',   // Modern UI text
      body: 'Inter'         // Readable body text
    },
    
    fontSize: {
      xs: 12,
      sm: 14,
      base: 16,
      lg: 18,
      xl: 20,
      '2xl': 24,
      '3xl': 30,
      '4xl': 36,
      '5xl': 48
    },
    
    fontWeight: {
      light: '300',
      normal: '400',
      medium: '500',
      semibold: '600',
      bold: '700',
      black: '900'
    },
    
    lineHeight: {
      tight: 1.2,
      normal: 1.5,
      relaxed: 1.6
    }
  },
  
  // Spacing Scale
  spacing: {
    0: 0,
    1: 4,
    2: 8,
    3: 12,
    4: 16,
    5: 20,
    6: 24,
    8: 32,
    10: 40,
    12: 48,
    16: 64,
    20: 80,
    24: 96
  },
  
  // Border Radius
  borderRadius: {
    none: 0,
    sm: 4,
    base: 8,
    md: 12,
    lg: 16,
    xl: 20,
    full: 9999
  },
  
  // Shadows and Glows
  shadows: {
    sm: '0 2px 4px rgba(0, 0, 0, 0.1)',
    base: '0 4px 8px rgba(0, 0, 0, 0.15)',
    lg: '0 8px 24px rgba(0, 0, 0, 0.2)',
    xl: '0 16px 48px rgba(0, 0, 0, 0.25)',
    glow: {
      cyan: '0 0 20px rgba(0, 212, 255, 0.5)',
      pink: '0 0 20px rgba(255, 0, 110, 0.5)',
      green: '0 0 20px rgba(57, 255, 20, 0.5)',
      purple: '0 0 20px rgba(168, 85, 247, 0.5)'
    }
  }
} as const;
```

### 1.2 Glassmorphism Utility Functions

```typescript
// Glassmorphism Style Generator
export const createGlassMorphism = (
  opacity: number = 0.1,
  blur: number = 20,
  borderOpacity: number = 0.2
) => ({
  backgroundColor: `rgba(255, 255, 255, ${opacity})`,
  backdropFilter: `blur(${blur}px)`,
  WebkitBackdropFilter: `blur(${blur}px)`, // Safari support
  borderWidth: 1,
  borderColor: `rgba(255, 255, 255, ${borderOpacity})`,
  borderStyle: 'solid'
});

// Neon Glow Effect Generator
export const createNeonGlow = (
  color: string,
  intensity: number = 0.5,
  size: number = 20
) => ({
  shadowColor: color,
  shadowOffset: { width: 0, height: 0 },
  shadowOpacity: intensity,
  shadowRadius: size,
  elevation: size / 2 // Android
});

// Gradient Generator
export const createGradient = (
  colors: string[],
  direction: 'horizontal' | 'vertical' | 'diagonal' = 'vertical'
) => {
  const start = direction === 'horizontal' ? { x: 0, y: 0 } :
                direction === 'vertical' ? { x: 0, y: 0 } :
                { x: 0, y: 0 };
                
  const end = direction === 'horizontal' ? { x: 1, y: 0 } :
              direction === 'vertical' ? { x: 0, y: 1 } :
              { x: 1, y: 1 };
              
  return {
    colors,
    start,
    end
  };
};
```

---

## 2. Core UI Components

### 2.1 Glass Panel Component

```typescript
import React from 'react';
import { View, ViewStyle, StyleSheet } from 'react-native';
import { LinearGradient } from 'expo-linear-gradient';

interface GlassPanelProps {
  children: React.ReactNode;
  style?: ViewStyle;
  opacity?: number;
  blur?: number;
  borderOpacity?: number;
  glowColor?: string;
  glowIntensity?: number;
}

export const GlassPanel: React.FC<GlassPanelProps> = ({
  children,
  style,
  opacity = 0.1,
  blur = 20,
  borderOpacity = 0.2,
  glowColor,
  glowIntensity = 0.3
}) => {
  const glassStyle = createGlassMorphism(opacity, blur, borderOpacity);
  const glowStyle = glowColor ? createNeonGlow(glowColor, glowIntensity) : {};
  
  return (
    <View style={[styles.glassPanel, glassStyle, glowStyle, style]}>
      <LinearGradient
        colors={['rgba(255,255,255,0.1)', 'rgba(255,255,255,0.05)']}
        style={styles.gradientOverlay}
      />
      <View style={styles.content}>
        {children}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  glassPanel: {
    borderRadius: GALAXY_THEME.borderRadius.lg,
    overflow: 'hidden',
    position: 'relative'
  },
  gradientOverlay: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0
  },
  content: {
    position: 'relative',
    zIndex: 1
  }
});
```

### 2.2 Neon Button Component

```typescript
import React from 'react';
import { 
  TouchableOpacity, 
  Text, 
  ViewStyle, 
  TextStyle,
  GestureResponderEvent
} from 'react-native';
import { LinearGradient } from 'expo-linear-gradient';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming
} from 'react-native-reanimated';

interface NeonButtonProps {
  title: string;
  onPress: (event: GestureResponderEvent) => void;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  glowColor?: string;
  disabled?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
}

const AnimatedTouchableOpacity = Animated.createAnimatedComponent(TouchableOpacity);

export const NeonButton: React.FC<NeonButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  size = 'md',
  glowColor = GALAXY_THEME.colors.neon.cyan,
  disabled = false,
  style,
  textStyle
}) => {
  const scale = useSharedValue(1);
  const glowIntensity = useSharedValue(0.3);
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
    ...createNeonGlow(glowColor, glowIntensity.value)
  }));
  
  const handlePressIn = () => {
    scale.value = withSpring(0.95);
    glowIntensity.value = withTiming(0.6, { duration: 150 });
  };
  
  const handlePressOut = () => {
    scale.value = withSpring(1);
    glowIntensity.value = withTiming(0.3, { duration: 300 });
  };
  
  const getButtonStyle = () => {
    const baseStyle = styles[`button_${size}`];
    const variantStyle = styles[`button_${variant}`];
    
    return [baseStyle, variantStyle, style];
  };
  
  const getTextStyle = () => {
    const baseTextStyle = styles[`text_${size}`];
    const variantTextStyle = styles[`text_${variant}`];
    
    return [baseTextStyle, variantTextStyle, textStyle];
  };
  
  return (
    <AnimatedTouchableOpacity
      style={[animatedStyle, getButtonStyle()]}
      onPress={onPress}
      onPressIn={handlePressIn}
      onPressOut={handlePressOut}
      disabled={disabled}
      activeOpacity={0.8}
    >
      {variant === 'primary' && (
        <LinearGradient
          colors={[glowColor, `${glowColor}80`]}
          style={styles.gradientBackground}
        />
      )}
      
      <Text style={getTextStyle()}>
        {title}
      </Text>
    </AnimatedTouchableOpacity>
  );
};

const styles = StyleSheet.create({
  // Button Sizes
  button_sm: {
    paddingHorizontal: GALAXY_THEME.spacing[3],
    paddingVertical: GALAXY_THEME.spacing[2],
    borderRadius: GALAXY_THEME.borderRadius.base,
    minWidth: 80
  },
  button_md: {
    paddingHorizontal: GALAXY_THEME.spacing[5],
    paddingVertical: GALAXY_THEME.spacing[3],
    borderRadius: GALAXY_THEME.borderRadius.md,
    minWidth: 120
  },
  button_lg: {
    paddingHorizontal: GALAXY_THEME.spacing[6],
    paddingVertical: GALAXY_THEME.spacing[4],
    borderRadius: GALAXY_THEME.borderRadius.lg,
    minWidth: 160
  },
  
  // Button Variants
  button_primary: {
    ...createGlassMorphism(0.2, 15, 0.3),
    borderWidth: 2
  },
  button_secondary: {
    ...createGlassMorphism(0.1, 10, 0.2),
    borderWidth: 1
  },
  button_ghost: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: 'rgba(255,255,255,0.3)'
  },
  
  // Text Sizes
  text_sm: {
    fontSize: GALAXY_THEME.typography.fontSize.sm,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    fontWeight: GALAXY_THEME.typography.fontWeight.medium
  },
  text_md: {
    fontSize: GALAXY_THEME.typography.fontSize.base,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    fontWeight: GALAXY_THEME.typography.fontWeight.semibold
  },
  text_lg: {
    fontSize: GALAXY_THEME.typography.fontSize.lg,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    fontWeight: GALAXY_THEME.typography.fontWeight.bold
  },
  
  // Text Variants
  text_primary: {
    color: GALAXY_THEME.colors.ui.white,
    textAlign: 'center'
  },
  text_secondary: {
    color: GALAXY_THEME.colors.ui.gray200,
    textAlign: 'center'
  },
  text_ghost: {
    color: GALAXY_THEME.colors.ui.gray300,
    textAlign: 'center'
  },
  
  gradientBackground: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    borderRadius: GALAXY_THEME.borderRadius.md
  }
});
```

### 2.3 Planet Card Component

```typescript
import React from 'react';
import { View, Text, Image, Dimensions } from 'react-native';
import { LinearGradient } from 'expo-linear-gradient';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
  Easing
} from 'react-native-reanimated';
import { GlassPanel } from './GlassPanel';
import { NeonButton } from './NeonButton';

interface PlanetCardProps {
  tool: AITool;
  onOpen: () => void;
  onFavorite: () => void;
  onShare: () => void;
  visible: boolean;
}

const { width: SCREEN_WIDTH } = Dimensions.get('window');
const CARD_WIDTH = SCREEN_WIDTH * 0.9;

export const PlanetCard: React.FC<PlanetCardProps> = ({
  tool,
  onOpen,
  onFavorite,
  onShare,
  visible
}) => {
  const translateY = useSharedValue(300);
  const opacity = useSharedValue(0);
  const scale = useSharedValue(0.8);
  
  React.useEffect(() => {
    if (visible) {
      translateY.value = withSpring(0, {
        damping: 20,
        stiffness: 100
      });
      opacity.value = withTiming(1, {
        duration: 400,
        easing: Easing.out(Easing.cubic)
      });
      scale.value = withSpring(1, {
        damping: 15,
        stiffness: 150
      });
    } else {
      translateY.value = withTiming(300, { duration: 300 });
      opacity.value = withTiming(0, { duration: 200 });
      scale.value = withTiming(0.8, { duration: 200 });
    }
  }, [visible]);
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateY: translateY.value },
      { scale: scale.value }
    ],
    opacity: opacity.value
  }));
  
  const getCategoryColor = (category: string) => {
    const categoryColors = {
      chat: GALAXY_THEME.colors.neon.cyan,
      image_generation: GALAXY_THEME.colors.neon.pink,
      coding: GALAXY_THEME.colors.neon.green,
      music_audio: GALAXY_THEME.colors.neon.gold,
      video_animation: GALAXY_THEME.colors.neon.purple,
      // Add more categories...
    };
    
    return categoryColors[category] || GALAXY_THEME.colors.neon.cyan;
  };
  
  const categoryColor = getCategoryColor(tool.category);
  
  return (
    <Animated.View style={[styles.container, animatedStyle]}>
      <GlassPanel
        style={styles.card}
        opacity={0.15}
        blur={25}
        glowColor={categoryColor}
        glowIntensity={0.4}
      >
        {/* Header with logo and close button */}
        <View style={styles.header}>
          <View style={styles.logoContainer}>
            <Image
              source={{ uri: tool.logoUrl }}
              style={styles.logo}
              resizeMode="contain"
            />
            <View style={styles.titleContainer}>
              <Text style={styles.title}>{tool.name}</Text>
              <Text style={styles.category}>{tool.category.replace('_', ' ')}</Text>
            </View>
          </View>
          
          <View style={styles.rating}>
            <Text style={styles.ratingText}>⭐ {tool.rating?.toFixed(1) || 'N/A'}</Text>
          </View>
        </View>
        
        {/* Description */}
        <Text style={styles.description} numberOfLines={3}>
          {tool.description}
        </Text>
        
        {/* Tags */}
        <View style={styles.tagsContainer}>
          {tool.tags?.slice(0, 3).map((tag, index) => (
            <View key={index} style={[styles.tag, { borderColor: categoryColor }]}>
              <Text style={styles.tagText}>{tag}</Text>
            </View>
          ))}
        </View>
        
        {/* Metrics */}
        <View style={styles.metrics}>
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {tool.monthlyUsers ? `${tool.monthlyUsers / 1000}k` : 'N/A'}
            </Text>
            <Text style={styles.metricLabel}>Users</Text>
          </View>
          
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {tool.reviewCount || 0}
            </Text>
            <Text style={styles.metricLabel}>Reviews</Text>
          </View>
          
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {tool.pricingModel || 'Free'}
            </Text>
            <Text style={styles.metricLabel}>Pricing</Text>
          </View>
        </View>
        
        {/* Action Buttons */}
        <View style={styles.actions}>
          <NeonButton
            title="🌐 Open Tool"
            onPress={onOpen}
            variant="primary"
            size="md"
            glowColor={categoryColor}
            style={styles.primaryButton}
          />
          
          <View style={styles.secondaryActions}>
            <NeonButton
              title="⭐"
              onPress={onFavorite}
              variant="ghost"
              size="sm"
              glowColor={GALAXY_THEME.colors.neon.gold}
              style={styles.iconButton}
            />
            
            <NeonButton
              title="📤"
              onPress={onShare}
              variant="ghost"
              size="sm"
              glowColor={GALAXY_THEME.colors.neon.cyan}
              style={styles.iconButton}
            />
          </View>
        </View>
        
        {/* Decorative gradient */}
        <LinearGradient
          colors={[`${categoryColor}20`, 'transparent']}
          style={styles.decorativeGradient}
          start={{ x: 0, y: 0 }}
          end={{ x: 1, y: 1 }}
        />
      </GlassPanel>
    </Animated.View>
  );
};

const styles = StyleSheet.create({
  container: {
    position: 'absolute',
    bottom: 100,
    left: (SCREEN_WIDTH - CARD_WIDTH) / 2,
    width: CARD_WIDTH,
    zIndex: 1000
  },
  card: {
    padding: GALAXY_THEME.spacing[5],
    borderRadius: GALAXY_THEME.borderRadius.xl
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'flex-start',
    marginBottom: GALAXY_THEME.spacing[4]
  },
  logoContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    flex: 1
  },
  logo: {
    width: 48,
    height: 48,
    borderRadius: GALAXY_THEME.borderRadius.md,
    marginRight: GALAXY_THEME.spacing[3]
  },
  titleContainer: {
    flex: 1
  },
  title: {
    fontSize: GALAXY_THEME.typography.fontSize.xl,
    fontFamily: GALAXY_THEME.typography.fontFamily.primary,
    fontWeight: GALAXY_THEME.typography.fontWeight.bold,
    color: GALAXY_THEME.colors.ui.white,
    marginBottom: GALAXY_THEME.spacing[1]
  },
  category: {
    fontSize: GALAXY_THEME.typography.fontSize.sm,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    color: GALAXY_THEME.colors.ui.gray300,
    textTransform: 'capitalize'
  },
  rating: {
    backgroundColor: 'rgba(255,255,255,0.1)',
    paddingHorizontal: GALAXY_THEME.spacing[2],
    paddingVertical: GALAXY_THEME.spacing[1],
    borderRadius: GALAXY_THEME.borderRadius.base
  },
  ratingText: {
    fontSize: GALAXY_THEME.typography.fontSize.sm,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    color: GALAXY_THEME.colors.ui.white
  },
  description: {
    fontSize: GALAXY_THEME.typography.fontSize.base,
    fontFamily: GALAXY_THEME.typography.fontFamily.body,
    color: GALAXY_THEME.colors.ui.gray200,
    lineHeight: GALAXY_THEME.typography.lineHeight.relaxed,
    marginBottom: GALAXY_THEME.spacing[4]
  },
  tagsContainer: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    marginBottom: GALAXY_THEME.spacing[4]
  },
  tag: {
    borderWidth: 1,
    borderRadius: GALAXY_THEME.borderRadius.full,
    paddingHorizontal: GALAXY_THEME.spacing[2],
    paddingVertical: GALAXY_THEME.spacing[1],
    marginRight: GALAXY_THEME.spacing[2],
    marginBottom: GALAXY_THEME.spacing[1],
    backgroundColor: 'rgba(255,255,255,0.05)'
  },
  tagText: {
    fontSize: GALAXY_THEME.typography.fontSize.xs,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    color: GALAXY_THEME.colors.ui.gray300
  },
  metrics: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: GALAXY_THEME.spacing[5],
    paddingVertical: GALAXY_THEME.spacing[3],
    borderTopWidth: 1,
    borderBottomWidth: 1,
    borderColor: 'rgba(255,255,255,0.1)'
  },
  metric: {
    alignItems: 'center'
  },
  metricValue: {
    fontSize: GALAXY_THEME.typography.fontSize.lg,
    fontFamily: GALAXY_THEME.typography.fontFamily.primary,
    fontWeight: GALAXY_THEME.typography.fontWeight.bold,
    color: GALAXY_THEME.colors.ui.white,
    marginBottom: GALAXY_THEME.spacing[1]
  },
  metricLabel: {
    fontSize: GALAXY_THEME.typography.fontSize.xs,
    fontFamily: GALAXY_THEME.typography.fontFamily.secondary,
    color: GALAXY_THEME.colors.ui.gray400
  },
  actions: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between'
  },
  primaryButton: {
    flex: 1,
    marginRight: GALAXY_THEME.spacing[3]
  },
  secondaryActions: {
    flexDirection: 'row'
  },
  iconButton: {
    width: 44,
    height: 44,
    marginLeft: GALAXY_THEME.spacing[2]
  },
  decorativeGradient: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    borderRadius: GALAXY_THEME.borderRadius.xl,
    opacity: 0.3,
    zIndex: -1
  }
});

// TypeScript interfaces
interface AITool {
  id: string;
  name: string;
  description: string;
  category: string;
  logoUrl?: string;
  rating?: number;
  reviewCount?: number;
  monthlyUsers?: number;
  pricingModel?: string;
  tags?: string[];
}
```

### 2.4 Cosmic Loader Component

```typescript
import React, { useEffect } from 'react';
import { View, Dimensions } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withRepeat,
  withTiming,
  withSequence,
  Easing
} from 'react-native-reanimated';
import { LinearGradient } from 'expo-linear-gradient';

interface CosmicLoaderProps {
  size?: number;
  colors?: string[];
  speed?: number;
}

const { width: SCREEN_WIDTH } = Dimensions.get('window');

export const CosmicLoader: React.FC<CosmicLoaderProps> = ({
  size = 80,
  colors = [GALAXY_THEME.colors.neon.cyan, GALAXY_THEME.colors.neon.pink, GALAXY_THEME.colors.neon.purple],
  speed = 2000
}) => {
  const rotation = useSharedValue(0);
  const scale = useSharedValue(1);
  const opacity = useSharedValue(0.6);
  
  useEffect(() => {
    // Continuous rotation
    rotation.value = withRepeat(
      withTiming(360, {
        duration: speed,
        easing: Easing.linear
      }),
      -1
    );
    
    // Pulsing scale effect
    scale.value = withRepeat(
      withSequence(
        withTiming(1.2, { duration: speed / 4 }),
        withTiming(0.8, { duration: speed / 4 }),
        withTiming(1.0, { duration: speed / 2 })
      ),
      -1
    );
    
    // Breathing opacity
    opacity.value = withRepeat(
      withSequence(
        withTiming(1, { duration: speed / 3 }),
        withTiming(0.4, { duration: speed / 3 }),
        withTiming(0.8, { duration: speed / 3 })
      ),
      -1
    );
  }, []);
  
  const outerRingStyle = useAnimatedStyle(() => ({
    transform: [
      { rotate: `${rotation.value}deg` },
      { scale: scale.value }
    ],
    opacity: opacity.value
  }));
  
  const middleRingStyle = useAnimatedStyle(() => ({
    transform: [
      { rotate: `${-rotation.value * 0.7}deg` },
      { scale: scale.value * 0.8 }
    ],
    opacity: opacity.value * 0.8
  }));
  
  const innerRingStyle = useAnimatedStyle(() => ({
    transform: [
      { rotate: `${rotation.value * 0.5}deg` },
      { scale: scale.value * 0.6 }
    ],
    opacity: opacity.value * 0.6
  }));
  
  return (
    <View style={styles.container}>
      {/* Outer Ring */}
      <Animated.View style={[outerRingStyle, { width: size, height: size }]}>
        <LinearGradient
          colors={[colors[0], colors[1]]}
          style={[styles.ring, styles.outerRing]}
          start={{ x: 0, y: 0 }}
          end={{ x: 1, y: 1 }}
        />
      </Animated.View>
      
      {/* Middle Ring */}
      <Animated.View style={[middleRingStyle, { width: size * 0.7, height: size * 0.7 }]}>
        <LinearGradient
          colors={[colors[1], colors[2]]}
          style={[styles.ring, styles.middleRing]}
          start={{ x: 0, y: 0 }}
          end={{ x: 1, y: 1 }}
        />
      </Animated.View>
      
      {/* Inner Ring */}
      <Animated.View style={[innerRingStyle, { width: size * 0.4, height: size * 0.4 }]}>
        <LinearGradient
          colors={[colors[2], colors[0]]}
          style={[styles.ring, styles.innerRing]}
          start={{ x: 0, y: 0 }}
          end={{ x: 1, y: 1 }}
        />
      </Animated.View>
      
      {/* Center Glow */}
      <View style={[styles.centerGlow, { width: size * 0.2, height: size * 0.2 }]} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
    justifyContent: 'center',
    position: 'relative'
  },
  ring: {
    position: 'absolute',
    borderRadius: 9999,
    borderWidth: 2,
    borderColor: 'transparent'
  },
  outerRing: {
    width: '100%',
    height: '100%',
    borderWidth: 3
  },
  middleRing: {
    width: '100%',
    height: '100%',
    borderWidth: 2
  },
  innerRing: {
    width: '100%',
    height: '100%',
    borderWidth: 2
  },
  centerGlow: {
    position: 'absolute',
    backgroundColor: GALAXY_THEME.colors.neon.cyan,
    borderRadius: 9999,
    opacity: 0.8,
    shadowColor: GALAXY_THEME.colors.neon.cyan,
    shadowOffset: { width: 0, height: 0 },
    shadowOpacity: 1,
    shadowRadius: 10,
    elevation: 10
  }
});
```

### 2.5 Star Field Background Component

```typescript
import React, { useEffect } from 'react';
import { View, Dimensions } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withRepeat,
  withTiming,
  withDelay,
  Easing
} from 'react-native-reanimated';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

interface Star {
  id: number;
  x: number;
  y: number;
  size: number;
  opacity: number;
  twinkleDelay: number;
}

interface StarFieldProps {
  starCount?: number;
  animationSpeed?: number;
}

export const StarField: React.FC<StarFieldProps> = ({
  starCount = 150,
  animationSpeed = 3000
}) => {
  const [stars] = React.useState<Star[]>(() =>
    Array.from({ length: starCount }, (_, i) => ({
      id: i,
      x: Math.random() * SCREEN_WIDTH,
      y: Math.random() * SCREEN_HEIGHT,
      size: Math.random() * 3 + 1,
      opacity: Math.random() * 0.8 + 0.2,
      twinkleDelay: Math.random() * animationSpeed
    }))
  );
  
  return (
    <View style={styles.starField}>
      {stars.map(star => (
        <StarPoint
          key={star.id}
          star={star}
          animationSpeed={animationSpeed}
        />
      ))}
    </View>
  );
};

interface StarPointProps {
  star: Star;
  animationSpeed: number;
}

const StarPoint: React.FC<StarPointProps> = ({ star, animationSpeed }) => {
  const opacity = useSharedValue(star.opacity);
  const scale = useSharedValue(1);
  
  useEffect(() => {
    // Twinkling animation
    opacity.value = withDelay(
      star.twinkleDelay,
      withRepeat(
        withTiming(Math.random() * 0.3 + 0.1, {
          duration: animationSpeed + Math.random() * 1000,
          easing: Easing.inOut(Easing.sine)
        }),
        -1,
        true
      )
    );
    
    // Subtle scale variation
    scale.value = withDelay(
      star.twinkleDelay / 2,
      withRepeat(
        withTiming(Math.random() * 0.5 + 0.8, {
          duration: animationSpeed * 1.5,
          easing: Easing.inOut(Easing.quad)
        }),
        -1,
        true
      )
    );
  }, []);
  
  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ scale: scale.value }]
  }));
  
  return (
    <Animated.View
      style={[
        styles.star,
        {
          left: star.x,
          top: star.y,
          width: star.size,
          height: star.size
        },
        animatedStyle
      ]}
    />
  );
};

const styles = StyleSheet.create({
  starField: {
    position: 'absolute',
    top: 0,
    left: 0,
    width: SCREEN_WIDTH,
    height: SCREEN_HEIGHT,
    zIndex: -1
  },
  star: {
    position: 'absolute',
    backgroundColor: GALAXY_THEME.colors.ui.white,
    borderRadius: 9999,
    shadowColor: GALAXY_THEME.colors.ui.white,
    shadowOffset: { width: 0, height: 0 },
    shadowOpacity: 0.8,
    shadowRadius: 2,
    elevation: 2
  }
});
```

---

## 3. Layout Components

### 3.1 Galaxy Screen Layout

```typescript
import React from 'react';
import { View, StatusBar, SafeAreaView } from 'react-native';
import { StarField } from './StarField';
import { GlassPanel } from './GlassPanel';

interface GalaxyScreenLayoutProps {
  children: React.ReactNode;
  showStarField?: boolean;
  statusBarStyle?: 'light-content' | 'dark-content';
}

export const GalaxyScreenLayout: React.FC<GalaxyScreenLayoutProps> = ({
  children,
  showStarField = true,
  statusBarStyle = 'light-content'
}) => {
  return (
    <View style={styles.container}>
      <StatusBar
        barStyle={statusBarStyle}
        backgroundColor="transparent"
        translucent
      />
      
      {showStarField && <StarField />}
      
      <SafeAreaView style={styles.safeArea}>
        {children}
      </SafeAreaView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: GALAXY_THEME.colors.cosmic.black
  },
  safeArea: {
    flex: 1
  }
});
```

### 3.2 Modal Layout with Glassmorphism

```typescript
import React from 'react';
import {
  Modal,
  View,
  TouchableWithoutFeedback,
  Dimensions,
  KeyboardAvoidingView,
  Platform
} from 'react-native';
import { BlurView } from 'expo-blur';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming
} from 'react-native-reanimated';
import { GlassPanel } from './GlassPanel';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

interface GalaxyModalProps {
  visible: boolean;
  onClose: () => void;
  children: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'full';
}

export const GalaxyModal: React.FC<GalaxyModalProps> = ({
  visible,
  onClose,
  children,
  size = 'md'
}) => {
  const opacity = useSharedValue(0);
  const scale = useSharedValue(0.8);
  const translateY = useSharedValue(50);
  
  React.useEffect(() => {
    if (visible) {
      opacity.value = withTiming(1, { duration: 300 });
      scale.value = withSpring(1, { damping: 20, stiffness: 100 });
      translateY.value = withSpring(0, { damping: 20, stiffness: 100 });
    } else {
      opacity.value = withTiming(0, { duration: 200 });
      scale.value = withTiming(0.8, { duration: 200 });
      translateY.value = withTiming(50, { duration: 200 });
    }
  }, [visible]);
  
  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [
      { scale: scale.value },
      { translateY: translateY.value }
    ]
  }));
  
  const getModalSize = () => {
    switch (size) {
      case 'sm':
        return { width: SCREEN_WIDTH * 0.8, maxHeight: SCREEN_HEIGHT * 0.5 };
      case 'md':
        return { width: SCREEN_WIDTH * 0.9, maxHeight: SCREEN_HEIGHT * 0.7 };
      case 'lg':
        return { width: SCREEN_WIDTH * 0.95, maxHeight: SCREEN_HEIGHT * 0.85 };
      case 'full':
        return { width: SCREEN_WIDTH, height: SCREEN_HEIGHT };
    }
  };
  
  return (
    <Modal
      visible={visible}
      transparent
      animationType="none"
      statusBarTranslucent
    >
      <View style={styles.modalOverlay}>
        <BlurView style={styles.blurOverlay} intensity={20} tint="dark">
          <TouchableWithoutFeedback onPress={onClose}>
            <View style={styles.backdropPressable} />
          </TouchableWithoutFeedback>
          
          <KeyboardAvoidingView
            behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
            style={styles.keyboardAvoidingView}
          >
            <Animated.View style={[animatedStyle, styles.modalContent, getModalSize()]}>
              <GlassPanel
                style={styles.modalPanel}
                opacity={0.15}
                blur={30}
                borderOpacity={0.3}
              >
                {children}
              </GlassPanel>
            </Animated.View>
          </KeyboardAvoidingView>
        </BlurView>
      </View>
    </Modal>
  );
};

const styles = StyleSheet.create({
  modalOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.5)'
  },
  blurOverlay: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  backdropPressable: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0
  },
  keyboardAvoidingView: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  modalContent: {
    maxWidth: SCREEN_WIDTH * 0.95,
    borderRadius: GALAXY_THEME.borderRadius.xl
  },
  modalPanel: {
    padding: GALAXY_THEME.spacing[6],
    borderRadius: GALAXY_THEME.borderRadius.xl
  }
});
```

This comprehensive UI component library provides all the necessary building blocks for creating the AI GALAXY app with consistent glassmorphism styling, smooth animations, and a cohesive cosmic aesthetic. Each component is designed to be performant, accessible, and visually stunning while maintaining the futuristic space theme throughout the application.