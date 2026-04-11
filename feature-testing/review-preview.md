"use client";
import { ProductRatingStar } from "@/components/product/product-rating-star";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { format } from "date-fns";
import Image from "next/image";
import placeholderImage from "./product-placeholder.jpeg";
import RatingAndReviews from "./rating-and-reviews";
import { useState } from "react";
import { getProductReviewList } from "@/lib/actions/product";
import DynamicPagination from "@/components/reusable/pagination";
import ProductReviewSkeleton from "@/components/skeleton/product-review";
import MediaPreview, {
  type MediaItem,
} from "@/components/reusable/media-preview";
import {
  IconBuildingStore,
  IconCircleCheckFilled,
  IconPlayerPlayFilled,
} from "@tabler/icons-react";

const buildMediaList = (pr: any): MediaItem[] => {
  const items: MediaItem[] = [];
  if (Array.isArray(pr?.images)) {
    pr.images.forEach((url: string) => items.push({ url, type: "image" }));
  }
  if (Array.isArray(pr?.videos)) {
    pr.videos.forEach((url: string) => items.push({ url, type: "video" }));
  }
  return items;
};

const ProductReviewComponent = ({
  data,
  productId,
}: {
  data: any;
  productId: string;
}) => {
  const [meta, setMeta] = useState<any>(data?.meta);
  const [productReviews, setProductReviews] = useState<any>(data?.resource);
  const [loading, setLoading] = useState(false);

  /* ---- Media preview state ---- */
  const [previewOpen, setPreviewOpen] = useState(false);
  const [previewMedia, setPreviewMedia] = useState<MediaItem[]>([]);
  const [previewIndex, setPreviewIndex] = useState(0);

  const openPreview = (media: MediaItem[], index: number) => {
    setPreviewMedia(media);
    setPreviewIndex(index);
    setPreviewOpen(true);
  };

  const refetchReviews = async (page = 1) => {
    if (!productId) return;
    setLoading(true);
    const updated = await getProductReviewList(productId, {
      page,
    });
    setMeta(updated?.meta);
    setProductReviews(updated?.resource);
    setLoading(false);
  };

  /* ---- Shared media thumbnails renderer ---- */
  const renderMediaThumbnails = (pr: any, imgSize: string, gap: string) => {
    const mediaList = buildMediaList(pr);
    const hasMedia = mediaList.length > 0;

    if (!hasMedia) {
      return (
        <Image
          src={placeholderImage}
          alt="placeholder"
          width={90}
          height={90}
          className="rounded shrink-0 border"
        />
      );
    }

    return (
      <div className={`flex flex-wrap items-center ${gap}`}>
        {mediaList.map((item, i) =>
          item.type === "image" ? (
            <button
              key={`img-${i}`}
              type="button"
              onClick={() => openPreview(mediaList, i)}
              className={`${imgSize} rounded overflow-hidden shrink-0 border relative group cursor-pointer transition-all hover:ring-2 hover:ring-primary/40`}
            >
              <Image
                src={item.url}
                alt="review image"
                fill
                sizes="96px"
                className="object-cover object-center"
              />
              {/* Hover overlay */}
              <div className="absolute inset-0 bg-black/0 group-hover:bg-black/10 transition-colors" />
            </button>
          ) : (
            <button
              key={`vid-${i}`}
              type="button"
              onClick={() => openPreview(mediaList, i)}
              className={`${imgSize} rounded-md overflow-hidden shrink-0 border relative group cursor-pointer bg-black transition-all hover:ring-2 hover:ring-primary/40`}
            >
              <video
                src={item.url}
                className="w-full h-full object-cover opacity-70 group-hover:opacity-90 transition-opacity"
                muted
                preload="metadata"
              />
              <div className="absolute inset-0 flex items-center justify-center">
                <div className="w-8 h-8 rounded-full bg-white/25 backdrop-blur-sm border border-white/30 flex items-center justify-center group-hover:scale-110 group-hover:bg-white/35 transition-all duration-300">
                  <IconPlayerPlayFilled
                    size={14}
                    className="text-white ml-0.5"
                  />
                </div>
              </div>
            </button>
          ),
        )}
      </div>
    );
  };

  return (
    <>
      <RatingAndReviews ratings={productReviews} />

      {meta?.total_items !== 0 && !loading && (
        <Card className="rounded-sm">
          <CardHeader className="border-b">
            <CardTitle>{`Product Review (${meta?.total_items})`}</CardTitle>
          </CardHeader>
          <CardContent>
            {productReviews?.reviews?.map((pr: any, index: number) => (
              <div
                key={index}
                className={`${
                  index !== productReviews?.reviews?.length - 1
                    ? "border-b"
                    : ""
                } sm:pb-6 pb-4.5 ${index === 0 ? "pt-0" : "sm:pt-6 pt-4.5"}`}
              >
                <div className={`flex w-full sm:gap-3.5 gap-2.5`}>
                  <Avatar className="size-10">
                    <AvatarImage
                      src={"/user.svg"}
                      alt={pr?.customer_name || "Customer Image"}
                    />
                    <AvatarFallback>
                      {pr?.customer_name?.charAt(0)?.toUpperCase() || "C"}
                    </AvatarFallback>
                  </Avatar>
                  <div className="flex flex-col gap-2 w-full">
                    <div className="flex items-start justify-between w-full">
                      <div className="flex flex-col justify-between space-y-2">
                        <ProductRatingStar
                          size={18}
                          value={pr?.product_rating}
                        />
                        <span className="text-muted-foreground sm:leading-5 sm:text-xs text-sm font-medium">
                          {pr?.customer_name ? pr?.customer_name : "Anonymous"}
                        </span>{" "}
                      </div>
                      <div className="flex sm:flex-col sm:items-start items-center sm:text-sm text-xs text-muted-foreground gap-2">
                        <span>{format(pr?.created_at, "d MMM yyyy")}</span>
                      </div>
                    </div>
                    {/* Desktop media section */}
                    <div className="sm:block hidden space-y-4">
                      <div className="text-card-foreground/90 text-base leading-5 space-x-2">
                        <span>{pr?.content}</span>
                      </div>
                      {renderMediaThumbnails(pr, "size-22.5", "gap-2.5")}
                    </div>
                  </div>
                </div>
                {/* Mobile media section */}
                <div className="sm:hidden block mt-4">
                  <div className="text-card-foreground/90 text-sm space-x-1">
                    <span className="text-muted-foreground">Quality:</span>
                    <span>{pr?.content}</span>
                  </div>
                  <div className="mt-4">
                    {renderMediaThumbnails(pr, "size-20", "gap-1.5")}
                  </div>
                </div>

                {pr?.reply && (
                  <div className="mt-6 sm:ml-14">
                    <div className="group/reply relative flex gap-3 rounded border border-border/60 bg-muted/30 p-4">
                      <div className="shrink-0">
                        <div className="flex size-9 items-center justify-center rounded bg-primary/10 border border-primary/20">
                          <IconBuildingStore
                            size={16}
                            className="text-primary"
                            stroke={1.75}
                          />
                        </div>
                      </div>

                      <div className="flex-1 min-w-0 space-y-2">
                        <div className="flex flex-wrap items-center justify-between gap-2">
                          <div className="flex items-center gap-2">
                            <span className="text-sm font-semibold text-foreground">
                              Store Response
                            </span>

                            <span className="inline-flex items-center gap-1 rounded-md bg-emerald-50 px-2 py-0.5 text-[10px] font-semibold uppercase tracking-wide text-emerald-700 border border-emerald-200/70">
                              <IconCircleCheckFilled
                                size={10}
                                className="text-emerald-500"
                              />
                              Verified
                            </span>
                          </div>

                          {pr.replied_at && (
                            <time
                              dateTime={pr.replied_at}
                              className="text-xs text-muted-foreground"
                            >
                              {format(new Date(pr.replied_at), "d MMM yyyy")}
                            </time>
                          )}
                        </div>

                        <p className="text-sm leading-relaxed text-foreground/80 whitespace-pre-line">
                          {pr.reply}
                        </p>
                      </div>
                    </div>
                  </div>
                )}
              </div>
            ))}
            <DynamicPagination
              data={meta}
              className="mt-6"
              handlePageChange={(page) => refetchReviews(page)}
            />
          </CardContent>
        </Card>
      )}
      {loading && <ProductReviewSkeleton />}

      {/* Fullscreen Media Preview */}
      <MediaPreview
        media={previewMedia}
        initialIndex={previewIndex}
        open={previewOpen}
        onClose={() => setPreviewOpen(false)}
      />
    </>
  );
};

export default ProductReviewComponent;
