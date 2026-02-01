{
  "name": "#damus Cloud Workflow v2",
  "nodes": [
    {
      "id": "schedule",
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "position": [-600, -200],
      "parameters": {
        "rule": {
          "interval": [
            {
              "unit": "hours",
              "value": 24
            }
          ]
        }
      }
    },
    {
      "id": "http-nostr",
      "name": "Fetch #damus Posts",
      "type": "n8n-nodes-base.httpRequest",
      "position": [-400, -200],
      "parameters": {
        "method": "GET",
        "url": "https://api.nostr.band/v0/search?query=%23damus&kind=1&limit=50",
        "responseFormat": "json"
      }
    },
    {
      "id": "extract",
      "name": "Extract Content",
      "type": "n8n-nodes-base.function",
      "position": [-200, -200],
      "parameters": {
        "functionCode": "return items[0].json.events.map(e => ({ json: { content: e.content } }));"
      }
    },
    {
      "id": "aggregate",
      "name": "Aggregate Content",
      "type": "n8n-nodes-base.aggregate",
      "position": [0, -200],
      "parameters": {
        "fieldsToAggregate": {
          "fieldToAggregate": [
            {
              "fieldToAggregate": "content"
            }
          ]
        }
      }
    },
    {
      "id": "ai",
      "name": "#damus AI Summary",
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "position": [200, -200],
      "parameters": {
        "text": "=Summarize the main themes from these #damus posts in a clean report format:\\n{{ $json.content.toJsonString() }}",
        "promptType": "define"
      }
    },
    {
      "id": "telegram",
      "name": "Send Telegram",
      "type": "n8n-nodes-base.telegram",
      "position": [400, -300],
      "parameters": {
        "text": "={{ $json.text.slice(0,4000) }}",
        "chatId": "={{ $env.TELEGRAM_CHAT_ID }}"
      }
    },
    {
      "id": "gmail",
      "name": "Send Gmail",
      "type": "n8n-nodes-base.gmail",
      "position": [400, -100],
      "parameters": {
        "sendTo": "YOUR_EMAIL@gmail.com",
        "subject": "#damus Daily Report",
        "message": "={{ $json.text }}",
        "options": {
          "appendAttribution": false
        }
      }
    }
  ],
  "connections": {
    "Schedule Trigger": {
      "main": [[{ "node": "Fetch #damus Posts", "type": "main", "index": 0 }]]
    },
    "Fetch #damus Posts": {
      "main": [[{ "node": "Extract Content", "type": "main", "index": 0 }]]
    },
    "Extract Content": {
      "main": [[{ "node": "Aggregate Content", "type": "main", "index": 0 }]]
    },
    "Aggregate Content": {
      "main": [[{ "node": "#damus AI Summary", "type": "main", "index": 0 }]]
    },
    "#damus AI Summary": {
      "main": [
        [{ "node": "Send Telegram", "type": "main", "index": 0 }],
        [{ "node": "Send Gmail", "type": "main", "index": 0 }]
      ]
    }
  }
}
