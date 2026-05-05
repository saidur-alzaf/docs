```
export function useImageFallback(fallback = "/image-fallback.png") {
  const getSrc = (src?: string | null) => {
    if (!src || src.trim() === "") return fallback;
    return src;
  };

  const onError = (e: React.SyntheticEvent<HTMLImageElement>) => {
    e.currentTarget.src = fallback;
    e.currentTarget.srcset = "";
  };

  return { onError, getSrc };
}

// usage example: -------------------------------------------------------------------------------------
// const { onError, getSrc } = useImageFallback();                                                    |
// custom fallback image: const { onError, getSrc } = useImageFallback("/placeholder-product.png");   |
//                                                                                                    |
// <Image                                                                                             |
//  src={getSrc(src)}                                                                                 |
//  alt={alt || "Product Image"}                                                                      |
//  fill                                                                                              |
//  className="object-cover"                                                                          |
//  onError={onError}                                                                                 |
// />                                                                                                 |
// ----------------------------------------------------------------------------------------------------


```
