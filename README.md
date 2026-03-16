{
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "weeks",
              "triggerAtHour": 7
            }
          ]
        }
      },
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.3,
      "position": [
        -9856,
        -448
      ],
      "id": "e6d7b159-5871-4743-a400-1adc3ba5fa5c",
      "name": "Schedule Trigger"
    },
    {
      "parameters": {
        "query": "Ai adoption For Small Business",
        "options": {
          "topic": "news",
          "max_results": 3,
          "time_range": "week"
        }
      },
      "type": "@tavily/n8n-nodes-tavily.tavily",
      "typeVersion": 1,
      "position": [
        -9632,
        -352
      ],
      "id": "ca32d973-bba5-43ce-b80f-bee926a15689",
      "name": "Research",
      "credentials": {
        "tavilyApi": {
          "id": "4akzlSpHMJLsa3OE",
          "name": "Tavily account"
        }
      }
    },
    {
      "parameters": {
        "model": "arcee-ai/trinity-large-preview:free",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenRouter",
      "typeVersion": 1,
      "position": [
        -8288,
        256
      ],
      "id": "02afdd53-1f57-497b-aa16-dd16c02335a0",
      "name": "OpenRouter Chat Model",
      "credentials": {
        "openRouterApi": {
          "id": "eeeVXLDiwH7K91l8",
          "name": "OpenRouter account"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "// Get AI Agent output\nconst text = $input.first().json.output || \"\";\n\n// Split lines\nconst lines = text.split(\"\\n\").map(l => l.trim()).filter(l => l);\n\n// Extract title (first bold line)\nlet title = \"\";\nconst titleMatch = text.match(/\\*\\*(.*?)\\*\\*/);\nif (titleMatch) {\n  title = titleMatch[1];\n}\n\n// Extract topics\nlet topics = [];\nlines.forEach(line => {\n  if (line.startsWith(\"-\")) {\n    topics.push(line.replace(\"-\", \"\").trim());\n  }\n});\n\n// Remove title + topics section to create summary\nlet summary = text\n  .replace(/\\*\\*(.*?)\\*\\*/g, \"\")\n  .replace(/Key Topics:/gi, \"\")\n  .replace(/-\\s.*/g, \"\")\n  .trim();\n\n// Clean summary\nsummary = summary.replace(/\\s+/g, \" \").trim();\n\n// Return structured output\nreturn [\n  {\n    json: {\n      title: title,\n      summary: summary,\n      topics: topics\n    }\n  }\n];"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -8160,
        -448
      ],
      "id": "7afa2d0d-6ff1-4866-867b-8e8c73f342dc",
      "name": "Code in JavaScript"
    },
    {
      "parameters": {
        "fieldToSplitOut": "topics",
        "options": {}
      },
      "type": "n8n-nodes-base.splitOut",
      "typeVersion": 1,
      "position": [
        -7936,
        -448
      ],
      "id": "16ad7e70-6fbe-4c84-9f0e-554096156bab",
      "name": "Split Out"
    },
    {
      "parameters": {
        "query": "={{ $json.topics }}",
        "options": {
          "time_range": "month",
          "include_raw_content": true
        }
      },
      "type": "@tavily/n8n-nodes-tavily.tavily",
      "typeVersion": 1,
      "position": [
        -7712,
        -448
      ],
      "id": "b95b28b1-99a4-4510-b666-30044616e503",
      "name": "Research Topics",
      "credentials": {
        "tavilyApi": {
          "id": "4akzlSpHMJLsa3OE",
          "name": "Tavily account"
        }
      }
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Topic:  {{ $json.query }}\n\nResearch: {{ $json.results.map(item => JSON.stringify(item,null,2)).join('\\n\\n') }}",
        "options": {
          "systemMessage": "=\n# Overview\nYou are a professional newsletter section writer. Your only task is to write one standalone section of a newsletter.\n\n## Instructions\n-Always include a clear section heading followed by the section content.\n- Do not write an overall title, introduction, or conclusion.\n-Write in a professional, expert, and engaging tone suitable for a business newsletter.\n- If you reference facts, data, or quotes, you must cite your sources and provide the actual clickable URLs.\n- Do not invent citations-only include real, verifiable sources.\n- Keep the section concise, well-structured, and easy to read."
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        -7264,
        -448
      ],
      "id": "d6d1f146-a107-4907-b7be-927d214fbcfe",
      "name": "Section Writer Agent"
    },
    {
      "parameters": {
        "fieldsToAggregate": {
          "fieldToAggregate": [
            {
              "fieldToAggregate": "output"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.aggregate",
      "typeVersion": 1,
      "position": [
        -6240,
        -544
      ],
      "id": "cfd859ef-3862-4122-a1e7-31692207751c",
      "name": "Aggregate"
    },
    {
      "parameters": {
        "model": "arcee-ai/trinity-mini:free",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenRouter",
      "typeVersion": 1,
      "position": [
        -6080,
        -128
      ],
      "id": "ce487fef-57f4-48f6-9f63-81da97400f2c",
      "name": "OpenRouter Chat Model1",
      "credentials": {
        "openRouterApi": {
          "id": "eeeVXLDiwH7K91l8",
          "name": "OpenRouter account"
        }
      }
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Title: {{ $('Planning Agent').item.json.output }}\n\nSection :{{ $json.output.join(\"\\n\\n\\n\") }}",
        "hasOutputParser": true,
        "options": {
          "systemMessage": "=# Overview\nYou are an expert newsletter editor. Merge exactly three provided sections into a cohesive, email-ready HTML body with a holistic introduction and conclusion. Maintain a professional, concise tone for a business audience and preserve intended meaning. I\n\n## Goals\n-Refine clarity, flow, and consistency across all sections.\n-Keep total content output ≤ 1000 words (hard limit). Avoid fluff.\n\n## Structure (HTML only)\n1) <p> Introduction that frames the three topics and their relevance; reference today's date: {{\n$now.format('dd-MM-yyyy') }}.\n2) For each provided section:\n-<h2> Use or lightly adjust the given title.\n-Edited <p> content.\n- Inline, clickable citations near related\nclaims using <a href=\"https://...\">Source</a>.\nNever invent sources.\n3) <h3>Sources</h3><ul> A single consolidated\nlist:\nEach item: <li><a href=\"[URL]\">[Publication\nName] [Article Title]</a></li>\nInclude full URLs, deduplicate identical links, and alphabetize by Publication Name.\n4) <p> Conclusion tying threads together with implications or next steps.\n\n## Citation and Link Rules\n-Every factual claim relying on external information must include a real, verifiable, clickable URL.\n-Preserve any provided citations; standardize to the <a> format.\n-Omit unverifiable links rather than fabricate.\n-Use only basic HTML tags (<h2>, <h3>, <p>, <ul>, <li>, <a>). No images, scripts, styles, oг external CSS.\n\n## Input\n\n-You will receive three sections. Edit and integrate them without adding unrelated topics.\n\nOutput Format (return only this)\n\nSubject: One clear, specific subject line (≤ 80 characters)\nContent:\n[HTML body only as specifid above]"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        -6016,
        -544
      ],
      "id": "bc63c917-4701-433a-a101-61bffabe872f",
      "name": "Editor Agent"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "={{ $json.results.map(item => JSON.stringify(item,null,2)).join('\\n\\n') }}",
        "options": {
          "systemMessage": "=#Overview \nYou are an expert newletter  planner. You will receive Three  articles from the past week. Your job is to come up with a creative and fun title as well as the  main topics for this newsletter.\n\n##Instructions\nthe newsletter should flow nicely, feel informative, and holistic. The topic should be 3-5 words."
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        -9184,
        -352
      ],
      "id": "75eacf0d-891e-4e35-b708-91ea953b229d",
      "name": "Planning Agent"
    },
    {
      "parameters": {
        "jsCode": "// 1️⃣ Get all input items\nconst items = $input.all();\nconst output = [];\n\n// 2️⃣ Loop through items\nfor (const item of items) {\n    let data = {};\n\n    const raw = item.json.output;\n\n    // 3️⃣ Check if valid JSON string\n    if (typeof raw === \"string\") {\n        try {\n            if (raw.trim().startsWith(\"{\")) {\n                // JSON format\n                data = JSON.parse(raw);\n            } else {\n                // Plain text format - try simple parsing\n                const subjectMatch = raw.match(/Subject:\\s*(.*)/i);\n                const contentMatch = raw.match(/Content:\\s*([\\s\\S]*)/i);\n\n                data.subject = subjectMatch ? subjectMatch[1].trim() : null;\n                data.content = contentMatch ? contentMatch[1].trim() : null;\n            }\n        } catch (err) {\n            throw new Error(`Failed to parse output: ${err.message}`);\n        }\n    } else if (typeof raw === \"object\") {\n        data = raw; // Already object\n    } else {\n        throw new Error(\"Unknown output format\");\n    }\n\n    // 4️⃣ Validate required fields\n    if (!data.subject || typeof data.subject !== \"string\") {\n        throw new Error(\"Validation Error: 'subject' field is missing or not a string\");\n    }\n    if (!data.content || typeof data.content !== \"string\") {\n        throw new Error(\"Validation Error: 'content' field is missing or not a string\");\n    }\n\n    // 5️⃣ Push clean output\n    output.push({\n        json: {\n            subject: data.subject,\n            content: data.content\n        }\n    });\n}\n\n// 6️⃣ Return all processed items\nreturn output;\n"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -4992,
        -640
      ],
      "id": "8e8c35bb-628d-4602-b010-fdc2389dc794",
      "name": "Code in JavaScript1"
    },
    {
      "parameters": {
        "sendTo": "sunilkumar99740979@gmail.com",
        "subject": "={{ $json.subject }}",
        "message": "={{ $json.content }}",
        "options": {}
      },
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 2.2,
      "position": [
        -4544,
        -640
      ],
      "id": "2623f1f5-b296-4265-a42b-5c76b83f1ddf",
      "name": "Send a message",
      "webhookId": "9e9ced9b-a6f9-433b-9312-653741c12da3",
      "credentials": {
        "gmailOAuth2": {
          "id": "nvczeBDNBDvJ3mHI",
          "name": "Gmail account"
        }
      }
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -9408,
        -352
      ],
      "id": "2ff9aec7-1a35-46b2-bcfc-de0321204902",
      "name": "Wait",
      "webhookId": "89a1b94b-638c-4d94-af3a-e6bad8ba314f"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -8384,
        -448
      ],
      "id": "8c17ffee-f277-4b5c-b05a-bb2c44d8dede",
      "name": "Wait1",
      "webhookId": "a552961a-53b0-47fc-b23d-e35028219208"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -7488,
        -448
      ],
      "id": "aa4d5c79-2eca-4cd9-b239-75fb2852fe0f",
      "name": "Wait2",
      "webhookId": "48bf0c8d-c238-49a4-9416-e93416175034"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -6464,
        -544
      ],
      "id": "ea784f5f-e8f1-4b2e-9e38-ce2fb61147a6",
      "name": "Wait3",
      "webhookId": "85758410-d00f-47ba-bd5f-3aba70a45c3c"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -5216,
        -640
      ],
      "id": "1b8a03cf-fa85-40a1-aba3-510a31a1eb0c",
      "name": "Wait4",
      "webhookId": "0c2843a9-440f-41a0-aadf-08691d249234"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -4768,
        -640
      ],
      "id": "de8b3155-6124-4969-9656-c2a2bbd12b76",
      "name": "Wait5",
      "webhookId": "c374a830-7e71-4dff-a9bf-bd6119b8ba86"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [
        -9856,
        -256
      ],
      "id": "774546ed-2ad8-4e2f-928d-c6dbb055864a",
      "name": "When clicking ‘Execute workflow’"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "59f21242-7985-4a1d-b4fc-08fb9d11f60f",
              "name": "Planning Agent",
              "value": "={{ $json.output }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        -8608,
        -448
      ],
      "id": "d9a033d3-b21a-4c36-b3fc-9c93db20219e",
      "name": "Edit Fields"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "7fa3e082-1f02-456c-ba18-06d30e51169e",
              "name": "Section Writer Agent",
              "value": "={{ $json.output }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        -6688,
        -544
      ],
      "id": "c9907f10-2209-455e-969a-40aa80acdd6d",
      "name": "Edit Fields1"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "2488e17f-ac51-4467-a2f2-38b7f2b62776",
              "name": "Editor Agent",
              "value": "={{ $json.output }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        -5440,
        -640
      ],
      "id": "612c547a-e69f-4b5e-b2f6-73ea185a57e0",
      "name": "Edit Fields2"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.errorTrigger",
      "typeVersion": 1,
      "position": [
        -9856,
        -32
      ],
      "id": "b9490ac9-df84-4f96-8061-aad9895d396e",
      "name": "Error Trigger"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "f996f6c1-9880-4728-bfa2-61d93876aa1e",
              "name": "execution.id",
              "value": "={{ $json.execution.id }}",
              "type": "number"
            },
            {
              "id": "15685965-29c3-4026-bd0f-e8c40ee239d7",
              "name": "execution.url",
              "value": "={{ $json.execution.url }}",
              "type": "string"
            },
            {
              "id": "c53f33d7-61a8-4d37-a2da-baeaa169e139",
              "name": "execution.error",
              "value": "={{ $json.execution.error }}",
              "type": "object"
            },
            {
              "id": "e2cb4269-824e-45b9-b29c-e09d4de96239",
              "name": "$workflow.name",
              "value": "={{ $workflow.name }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        -9632,
        -32
      ],
      "id": "5324c710-a3a9-49b0-9654-7e94bbb15096",
      "name": "Edit Fields3"
    },
    {
      "parameters": {
        "chatId": "6159190784",
        "text": "Error in News Letter Agent",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        -9408,
        -32
      ],
      "id": "f4600082-11a3-4f78-830f-f74db5590b85",
      "name": "Send a text message",
      "webhookId": "206c20c0-d3fb-419d-865f-4a4a225f6f0e",
      "credentials": {
        "telegramApi": {
          "id": "HqVk0JGEQwsKo59i",
          "name": "Telegram account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "0213ba92-8fa4-4230-a478-0ad942e30aa9",
              "leftValue": "={{ $json.output }}",
              "rightValue": "",
              "operator": {
                "type": "string",
                "operation": "notEmpty",
                "singleValue": true
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        -8832,
        -352
      ],
      "id": "cf8fe3b1-1ba0-4f08-bb35-9a446f790917",
      "name": "If"
    },
    {
      "parameters": {
        "errorMessage": "Error in Planning Agent"
      },
      "type": "n8n-nodes-base.stopAndError",
      "typeVersion": 1,
      "position": [
        -8608,
        -256
      ],
      "id": "27b3e83c-3a7a-4fa4-ba63-f030cdce3603",
      "name": "Stop and Error"
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "0213ba92-8fa4-4230-a478-0ad942e30aa9",
              "leftValue": "={{ $json.output }}",
              "rightValue": "",
              "operator": {
                "type": "string",
                "operation": "notEmpty",
                "singleValue": true
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        -5664,
        -544
      ],
      "id": "371fef63-6477-4a2c-b7a9-d60080ee609e",
      "name": "If2"
    },
    {
      "parameters": {
        "errorMessage": "Error in Editor Agent"
      },
      "type": "n8n-nodes-base.stopAndError",
      "typeVersion": 1,
      "position": [
        -5360,
        -192
      ],
      "id": "1d02e142-9e2e-4f92-9be1-f865fbb357fa",
      "name": "Stop and Error2"
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "0213ba92-8fa4-4230-a478-0ad942e30aa9",
              "leftValue": "={{ $json.output }}",
              "rightValue": "",
              "operator": {
                "type": "string",
                "operation": "notEmpty",
                "singleValue": true
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        -6912,
        -448
      ],
      "id": "5c130b1a-c5fb-49d9-a35e-a76da3a4ce50",
      "name": "If3"
    },
    {
      "parameters": {
        "errorMessage": "Error in Editor Agent"
      },
      "type": "n8n-nodes-base.stopAndError",
      "typeVersion": 1,
      "position": [
        -6688,
        -160
      ],
      "id": "9ac0162d-a501-4ca1-affe-1efd5fd4d754",
      "name": "Stop and Error3"
    },
    {
      "parameters": {
        "content": "",
        "height": 1312,
        "width": 5712,
        "color": 4
      },
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        -10016,
        -800
      ],
      "typeVersion": 1,
      "id": "a2cd999e-c119-42db-a87d-2fcc8d019577",
      "name": "Sticky Note"
    }
  ],
  "connections": {
    "Schedule Trigger": {
      "main": [
        [
          {
            "node": "Research",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Research": {
      "main": [
        [
          {
            "node": "Wait",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenRouter Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "Planning Agent",
            "type": "ai_languageModel",
            "index": 0
          },
          {
            "node": "Section Writer Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Code in JavaScript": {
      "main": [
        [
          {
            "node": "Split Out",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Split Out": {
      "main": [
        [
          {
            "node": "Research Topics",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Research Topics": {
      "main": [
        [
          {
            "node": "Wait2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Section Writer Agent": {
      "main": [
        [
          {
            "node": "If3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Aggregate": {
      "main": [
        [
          {
            "node": "Editor Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenRouter Chat Model1": {
      "ai_languageModel": [
        [
          {
            "node": "Editor Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Editor Agent": {
      "main": [
        [
          {
            "node": "If2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Planning Agent": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code in JavaScript1": {
      "main": [
        [
          {
            "node": "Wait5",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait": {
      "main": [
        [
          {
            "node": "Planning Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait1": {
      "main": [
        [
          {
            "node": "Code in JavaScript",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait2": {
      "main": [
        [
          {
            "node": "Section Writer Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait3": {
      "main": [
        [
          {
            "node": "Aggregate",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait4": {
      "main": [
        [
          {
            "node": "Code in JavaScript1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait5": {
      "main": [
        [
          {
            "node": "Send a message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "When clicking ‘Execute workflow’": {
      "main": [
        [
          {
            "node": "Research",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Edit Fields": {
      "main": [
        [
          {
            "node": "Wait1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Edit Fields1": {
      "main": [
        [
          {
            "node": "Wait3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Edit Fields2": {
      "main": [
        [
          {
            "node": "Wait4",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Error Trigger": {
      "main": [
        [
          {
            "node": "Edit Fields3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Edit Fields3": {
      "main": [
        [
          {
            "node": "Send a text message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Send a text message": {
      "main": [
        []
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "Edit Fields",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Stop and Error",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If2": {
      "main": [
        [
          {
            "node": "Edit Fields2",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Stop and Error2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If3": {
      "main": [
        [
          {
            "node": "Edit Fields1",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Stop and Error3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "86d81f670c962873ff00838d1f71a4d53552e327fd745f40e6a14070c5e01c4c"
  }
}
