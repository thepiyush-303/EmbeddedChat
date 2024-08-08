# Embedded Chat: A Staple in Excellent Customer Service

An easy-to-use, full-stack component (React.js + backend behaviors) for embedding Rocket.Chat into your web app.

![ec-demo-image](https://github.com/RocketChat/EmbeddedChat/assets/78961432/b85c7b8a-65e2-4a90-a843-f4072c942ac0)

## Technical Details on Theming

This guide dives into the technical aspects of how themes, styles, and configurations are implemented within the application.

## Understanding the Theming Approach

We use [Emotion](https://emotion.sh/) for styling. Each component has a `component.style.js` file for CSS definitions.

We use theming with our `useTheme` hook, exported from `@embeddedchat/ui-elements`, according to our specific requirements. The implementation looks something like this:

```jsx
import { useContext } from 'react';
import { ThemeContext } from '../context/ThemeContextProvider';
import DefaultTheme from '../theme/DefaultTheme';

const invertMode = (mode) => (mode === 'light' ? 'dark' : 'light');

const useTheme = () => {
  const context = useContext(ThemeContext);

  if (!context) {
    const defaultMode = 'light';
    const defaultTheme = DefaultTheme;
    const colors = defaultTheme.schemes?.[defaultMode];
    const invertedColors = defaultTheme.schemes?.[invertMode(defaultMode)];

    return {
      theme: defaultTheme,
      mode: defaultMode,
      colors,
      invertedColors,
      setMode: () => {},
      setTheme: () => {},
    };
  }

  return context;
};

export default useTheme;
```

This hook grants access to the theme, mode, colors, and invertedColors, streamlining their usage across different parts of the application. It ensures that colors are dynamically provided according to the current theme mode (light or dark), eliminating the need for repetitive code.

For example, in the CSS:

```jsx
const { colors } = useTheme();
const main = css`
  margin: 0.2rem 2rem;
  display: block;
  max-height: 10rem;
  overflow: scroll;
  overflow-x: hidden;
  border: 1px solid ${colors.border};
  border-radius: 0.2rem;
  color: ${colors.secondaryForeground};
`;
```

Here, `colors` from `useTheme` sets `colors.border` and `colors.secondaryForeground`.

## Technical Overview of `useComponentOverrides`

Components utilize the `useComponentOverrides` hook to facilitate various types of customizations:

### Example: styleOverrides and classNames

```jsx
import { useComponentOverrides } from '../../hooks/useComponentOverrides';

export const MessageBody = ({
  children,
  className = '',
  style = {},
  ...props
}) => {
  const { styleOverrides, classNames } = useComponentOverrides(
    'MessageBody',
    className,
    style
  );

  return (
    <Box
      css={MessageBodyCss}
      className={appendClassNames('ec-message-body', classNames)}
      style={styleOverrides}
      {...props}
    >
      <p>{children}</p>
    </Box>
  );
};
```

In this example, it's evident that the className and style properties are dynamically applied based on the values received from styleOverrides for styles and classNames for classes.

### Example: Config Overrides

```jsx
const { styleOverrides, classNames, configOverrides } = useComponentOverrides(
  'MessageToolbox',
  className,
  style
);

// Extract surfaceItems and menuItems from configOverrides
const surfaceItems = configOverrides.optionConfig?.surfaceItems || optionConfig.surfaceItems;
const menuItems = configOverrides.optionConfig?.menuItems || optionConfig.menuItems;

// Process surfaceItems and menuItems based on the configuration
const menuOptions = menuItems
  ?.map((item) => options[item]?.visible ? {
    id: options[item].id,
    action: options[item].onClick,
    label: options[item].label,
    icon: options[item].iconName,
  } : null)
  .filter((option) => option !== null);

const surfaceOptions = surfaceItems
  ?.map((item) => options[item]?.visible ? {
    id: options[item].id,
    onClick: options[item].onClick,
    label: options[item].label,
    iconName: options[item].iconName,
    type: options[item].type,
  } : null)
  .filter((option) => option !== null);

// Render the options
{surfaceOptions?.length > 0 && (
  <SurfaceMenu options={surfaceOptions} size="small" />
)}
{menuOptions?.length > 0 && (
  <Menu
    size="small"
    options={menuOptions}
    tooltip={{ isToolTip: true, position: 'top', text: 'More' }}
    useWrapper={false}
    style={{ top: 'auto', bottom: 'calc(100% + 2px)' }}
  />
)}
```

In this snippet, `surfaceItems` and `menuItems` are retrieved from `configOverrides.optionConfig` or fall back to default values (`optionConfig.surfaceItems` and `optionConfig.menuItems`) when not provided in `configOverrides`. These items are then processed and rendered accordingly.
### Example: Variant Overrides

```jsx
const { styleOverrides, classNames, variantOverrides } =
  useComponentOverrides('MessageHeader');

// Determine the display name variant or default to 'Normal'
const displayNameVariant = variantOverrides || 'Normal';

// Render a span element for the message header username
<Box
  is="span"
  css={styles.userName}
  className={appendClassNames('ec-message-header-username', classNames)}
  style={
    displayNameVariant === 'colorize'
      ? { color: getDisplayNameColor(message.u.username) }
      : null
  }
/>;
```

In this code snippet, variantOverrides provides customization options for the 'MessageHeader' component, influencing how the username is displayed based on the displayNameVariant.

## Applying Class Names for Easy Styling

Components like `MessageBody` are assigned specific class names (`ec-message-body`) for simplified CSS application.

## Conclusion

These insights enable straightforward customization of Embedded Chat's source code, ensuring adaptability to diverse project requirements.