# Image Optimization

Images are usually the heaviest part of any web page. A single unoptimized photo can be 5MB. That is more than your entire JavaScript bundle. I learned this the hard way when my app took 8 seconds to load because of hero images.

## Lazy Loading Images

Do not load images the user cannot see yet. The `loading="lazy"` attribute tells the browser to wait until the image is near the viewport:

```jsx
function ImageGallery({ images }) {
  return images.map((img) => (
    <img
      key={img.id}
      src={img.url}
      alt={img.description}
      loading="lazy"
      className="w-full rounded"
    />
  ));
}
```

Just one attribute and the browser handles the rest. This works in all modern browsers.

## Intersection Observer for More Control

When you need more control over when images load, use the Intersection Observer:

```jsx
function LazyImage({ src, alt }) {
  const [isVisible, setIsVisible] = useState(false);
  const imgRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsVisible(true);
        observer.disconnect();
      }
    });

    if (imgRef.current) observer.observe(imgRef.current);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={imgRef}>
      {isVisible ? (
        <img src={src} alt={alt} className="w-full rounded" />
      ) : (
        <div className="w-full h-48 bg-gray-200 rounded animate-pulse" />
      )}
    </div>
  );
}
```

## Blur Placeholder

Show a blurred low-resolution preview while the full image loads:

```jsx
function ImageWithPlaceholder({ src, placeholderSrc, alt }) {
  const [loaded, setLoaded] = useState(false);

  return (
    <div className="relative overflow-hidden rounded">
      {/* Blurred placeholder */}
      <img
        src={placeholderSrc}
        alt=""
        className={`w-full blur-lg scale-110 transition-opacity duration-500 ${
          loaded ? "opacity-0" : "opacity-100"
        }`}
      />
      {/* Full image */}
      <img
        src={src}
        alt={alt}
        onLoad={() => setLoaded(true)}
        className="w-full absolute inset-0"
      />
    </div>
  );
}
```

The placeholder is a tiny image (maybe 20px wide) that the browser loads instantly. It blurs to hide the low quality. When the full image loads, it fades in smoothly.

## WebP Format

WebP images are 25-35% smaller than JPEG at the same quality. Use the `picture` element for format fallback:

```jsx
function OptimizedImage({ webpSrc, fallbackSrc, alt }) {
  return (
    <picture>
      <source srcSet={webpSrc} type="image/webp" />
      <source srcSet={fallbackSrc} type="image/jpeg" />
      <img src={fallbackSrc} alt={alt} loading="lazy" />
    </picture>
  );
}
```

Browsers that support WebP use the first source. Others fall back to JPEG.

## Responsive Images

Serve different sizes for different screens. A phone does not need a 2000px wide image:

```jsx
function ResponsiveImage() {
  return (
    <img
      srcSet={`${small} 480w, ${medium} 800w, ${large} 1200w`}
      sizes="(max-width: 600px) 480px, (max-width: 900px) 800px, 1200px"
      src={medium}
      alt="Landscape photo"
      loading="lazy"
    />
  );
}
```

The browser picks the smallest image that fills the required size. A phone gets the 480px version, a desktop gets the 1200px version.

These four techniques together can cut your image payload by 70% or more. Lazy load, use WebP, serve the right size, and show a placeholder while loading. Your users on slow connections will notice the difference immediately.
