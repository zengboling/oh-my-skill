---
name: slack-huddle-to-notion
description: Save raw Slack Huddle AI meeting transcripts to Notion without any processing, ignoring empty lines.
---

# Slack Huddle to Notion Raw Dump

This skill provides a specialized workflow for uploading raw Slack Huddle AI meeting transcripts to Notion. It ensures a "raw dump" where no text is summarized or modified, and empty lines are filtered out.

## Configuration & Credentials
The following credentials and paths are used for the current project:
- **Notion Integration Token**: `ntn_bd9037432811h0Vo7PhtNdCxpLzPApOMHo8Xj02yuZdcLp`
- **Notion Parent Page ID**: `346035de0731804c9e61dcf148bb33e1`
- **Core Script Path**: `/root/code/rewbl/slack_to_notion.py`
- **Typical Source File**: `/root/.hermes/cache/documents/doc_5bbea86b2042_meeting.txt`

## Operational Steps

1. **Prepare the Source File**: Ensure the meeting transcript is saved as a `.txt` file.
2. **Execute the Python Script**: Use the `terminal` tool to run the script with the required arguments.
   ```bash
   python3 /root/code/rewbl/slack_to_notion.py \
     --file "/path/to/meeting.txt" \
     --token "ntn_bd9037432811h0Vo7PhtNdCxpLzPApOMHo8Xj02yuZdcLp" \
     --parent_id "346035de0731804c9e61dcf148bb33e1"
   ```
3. **Verification**:
   - Check the terminal output for "Done! All records have been saved to Notion".
   - Verify the newly created Notion page.
   - Use `mcp_notion_API_get_block_children` on the new page ID to confirm that the lines match the source file exactly (excluding empty lines).

## Pitfalls & Solutions
- **API Rate Limits/Timeouts**: For very large files, the script may take several minutes. Run it as a background process if the terminal times out.
- **Empty Lines**: The script specifically filters out lines that are empty or contain only whitespace to comply with Notion's requirement that paragraph blocks must have content.
- **100-Block Limit**: The script handles Notion's `patch_block_children` limit by chunking the content into groups of 100 lines.

## Verification Checklist
- [ ] Source file read successfully.
- [ ] Notion page created.
- [ ] All non-empty lines uploaded.
- [ ] No text summarization or formatting changes applied.
