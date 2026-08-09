---
title: "Detect ingredients with Amazon Rekognition"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### 1. Grant Rekognition access

The EC2 IAM role needs the minimum action:

```json
{
  "Effect": "Allow",
  "Action": "rekognition:DetectLabels",
  "Resource": "*"
}
```

The role also needs `s3:GetObject` for `arn:aws:s3:::my-foodie-ai-images/ai-images/*` because Rekognition reads the object by bucket and key.

#### 2. Invoke `DetectLabels`

In `api/src/common/aws/rekognition.service.ts`:

```ts
async detectLabels(imageKey: string) {
  return this.client.send(new DetectLabelsCommand({
    Image: {
      S3Object: {
        Bucket: this.configService.getOrThrow('AWS_BUCKET_NAME'),
        Name: imageKey,
      },
    },
    MaxLabels: 30,
    MinConfidence: 50,
  }));
}
```

`MinConfidence: 50` limits results returned by Rekognition; the Backend applies a stricter threshold when selecting ingredients.

#### 3. Normalize ingredient labels

`IngredientService` performs four steps:

1. Keep labels at `80%` confidence or higher.
2. Remove generic labels such as `Food`, `Meal`, `Dish`, `Plate`, `Fruit`, and `Vegetable`.
3. Normalize equivalent names, for example `Scallion` to `Green Onion`.
4. Deduplicate and sort by descending confidence.

```ts
labels
  .filter((label) => (label.Confidence ?? 0) >= 80)
  .filter((label) => !ignoreLabels.has(label.Name!))
  .map((label) => ({
    name: normalize(label.Name!),
    confidence: Number((label.Confidence ?? 0).toFixed(2)),
  }))
  .filter((item, index, self) =>
    self.findIndex((value) => value.name === item.name) === index,
  )
  .sort((a, b) => b.confidence - a.confidence);
```

#### 4. Actual result

The interface displays labels and confidence values returned by Rekognition:

![Labels detected by Rekognition](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.5-Week5/rekognition-labels-result.png)

Labels are stored with generation history so the AI input can be traced:

![Rekognition labels stored in generation history](/AWS-WorkBlog-LeHoaiLoc/images/1-Worklog/1.5-Week5/rekognition-labels-history.png)

#### 5. Errors and current limitations

| Case | Handling |
| ---- | -------- |
| The S3 object does not exist | Log contextual information and return a normalized error |
| Region or bucket is incorrect | Check `AWS_REGION`, `AWS_BUCKET_NAME`, and the IAM role |
| No suitable labels are found | Skip Bedrock or ask the user for a clearer image |
| Rekognition times out or throttles | Apply bounded retries with backoff |

> **Warning:** The current source does not invoke `DetectModerationLabels`. Do not claim that an image was moderated or rejected by Rekognition until the service, policy, and tests are implemented.

#### 6. Verify

1. Use a clear ingredient image and confirm confidence-bearing labels.
2. Use a non-food image and confirm that false ingredients are not generated.
3. Use a missing object key to verify error logging.
4. Confirm AI history stores the model, labels, status, and recipe ID.
