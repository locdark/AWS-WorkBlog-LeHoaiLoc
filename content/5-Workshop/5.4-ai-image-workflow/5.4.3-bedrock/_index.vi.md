---
title: "Tạo và lưu công thức với Amazon Bedrock"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### 1. Chọn model và cấp quyền

Khai báo model bằng `BEDROCK_MODEL_ID` và xác nhận model có sẵn trong Region. IAM Role của EC2 chỉ cần quyền gọi model được sử dụng:

```json
{
  "Effect": "Allow",
  "Action": "bedrock:InvokeModel",
  "Resource": "arn:aws:bedrock:<region>::foundation-model/<model-id>"
}
```

Không hard-code model ID trong service để có thể thay đổi model theo môi trường.

#### 2. Tạo prompt từ nguyên liệu

Bedrock không nhận ảnh trực tiếp trong implementation này. `PromptBuilderService` nhận danh sách nguyên liệu từ Rekognition và tạo prompt text yêu cầu đúng một JSON object:

```ts
const ingredientList = ingredients
  .map((item) => `- ${item.name} (${item.confidence}%)`)
  .join('\n');

return `
You are a professional chef.
Detected ingredients:
${ingredientList}

Requirements:
- Create ONE recipe.
- Write the recipe in Vietnamese.
- Return ONLY valid JSON without markdown.
- Preserve every sourceName from the input.
`;
```

Schema output gồm `detectedIngredients`, `title`, `description`, `cookTime`, `difficulty`, `servings`, `ingredients`, `steps`, `tips` và `nutrition`.

#### 3. Gọi Bedrock Runtime

`BedrockService` dùng Converse API:

```ts
const command = new ConverseCommand({
  modelId: config.getOrThrow('BEDROCK_MODEL_ID'),
  messages: [{
    role: ConversationRole.USER,
    content: [{ text: prompt }],
  }],
  inferenceConfig: {
    maxTokens: 1500,
    temperature: 0.7,
  },
});

const response = await client.send(command);
return response.output?.message?.content?.[0]?.text ?? '';
```

Trong lúc chờ response, giao diện khóa nút gửi để tránh tạo request trùng:

![Bedrock đang tạo công thức](/images/1-Worklog/1.6-Week6/bedrock-processing.png)

#### 4. Validate và lưu transaction

Backend parse JSON, kiểm tra các trường bắt buộc và map nguyên liệu được nhận diện sang tên tiếng Việt. Sau đó `RecipePersistenceService` lưu recipe, ingredients và steps trong một transaction. AI history ghi model, prompt, labels, recipe ID và trạng thái.

Nếu parse, validate hoặc transaction thất bại, history được ghi `FAILED` và API trả lỗi chuẩn hóa; không lưu một công thức dở dang.

#### 5. Kết quả

![Kết quả công thức được Bedrock tạo](/images/1-Worklog/1.6-Week6/bedrock-recipe-result.png)

![Nguyên liệu và các bước thực hiện đã được lưu](/images/1-Worklog/1.6-Week6/generated-recipe-details.png)

#### 6. Kiểm soát chất lượng và chi phí

- Không gọi Bedrock khi danh sách nguyên liệu rỗng.
- Giữ prompt ngắn, giới hạn `maxTokens` và số lần retry.
- Validate JSON trước khi mở transaction database.
- Log model ID, thời gian xử lý và history ID; không log ảnh/base64 hoặc secret.
- Theo dõi số request thành công/thất bại để phát hiện prompt hoặc model không ổn định.

#### 7. Kiểm tra

1. Gửi danh sách nguyên liệu hợp lệ và xác nhận recipe được lưu.
2. Mock output không phải JSON và xác nhận history `FAILED`.
3. Mock thiếu `steps` hoặc `ingredients` và xác nhận transaction rollback.
4. Kiểm tra response trả recipe ID, history ID, ingredients và thumbnail URL.
