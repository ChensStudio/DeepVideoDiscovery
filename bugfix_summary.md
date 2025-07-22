# Bug Fix Summary: ValueError: subtitle_source must be provided for local videos

## Problem Description

When running the command:
```bash
python local_run.py https://youtu.be/d2jPd-JPg6g "how many animals appear in this video"
```

The system encountered the following error:
```
ValueError: subtitle_source must be provided for local videos.
```

## Root Cause Analysis

The error was caused by incorrect parameter passing to the `load_video()` function in three files:

### Function Definition
```python
def load_video(
    video_source: str,           # Video URL or local path
    with_subtitle: bool = False, # Whether to download subtitles
    subtitle_source: str | None = None,  # Subtitle source
) -> str:
```

### Incorrect Function Calls
The following files had incorrect function calls:

1. **`local_run.py:48`**
   ```python
   load_video(video_url, video_path)  # ❌ Wrong: video_path interpreted as with_subtitle=True
   ```

2. **`mcp_server.py:48`**
   ```python
   load_video(video_url, video_path)  # ❌ Wrong: video_path interpreted as with_subtitle=True
   ```

3. **`app.py:55`**
   ```python
   load_video(video_url, video_path)  # ❌ Wrong: video_path interpreted as with_subtitle=True
   ```

### Why This Caused the Error

1. The second parameter `video_path` (a string) was being passed to the `with_subtitle` parameter (expecting a boolean)
2. Python interpreted the non-empty string as `True`
3. When `with_subtitle=True` and processing a local video, the function requires a `subtitle_source` parameter
4. Since no `subtitle_source` was provided, the function raised the ValueError

## Solution Applied

### Fixed Function Calls
All three files were corrected to call the function properly:

1. **`local_run.py:48`**
   ```python
   load_video(video_url)  # ✅ Correct: Only download video, no subtitles
   ```

2. **`mcp_server.py:48`**
   ```python
   load_video(video_url)  # ✅ Correct: Only download video, no subtitles
   ```

3. **`app.py:55`**
   ```python
   load_video(video_url)  # ✅ Correct: Only download video, no subtitles
   ```

### Why This Fix Works

1. The `load_video()` function automatically determines the output path based on the video ID extracted from the URL
2. It saves videos to `./video_database/raw/{video_id}.mp4` by default
3. The function returns the absolute path of the downloaded video file
4. No subtitle download is needed in these contexts since subtitles are handled separately

## Testing

After applying the fix, the command should run successfully:
```bash
python local_run.py https://youtu.be/d2jPd-JPg6g "how many animals appear in this video"
```

## Additional Improvements

1. **Updated Documentation**: Fixed the `dvd_command_flow_analysis.md` to reflect the correct default configuration (`LITE_MODE = False`)
2. **Consistency**: Ensured all three entry points (`local_run.py`, `app.py`, `mcp_server.py`) use the same correct function call pattern

## Files Modified

- ✅ `local_run.py` - Fixed load_video call
- ✅ `mcp_server.py` - Fixed load_video call  
- ✅ `app.py` - Fixed load_video call
- ✅ `dvd_command_flow_analysis.md` - Updated documentation
- ✅ `bugfix_summary.md` - Created this summary

This fix resolves the immediate error and ensures consistent behavior across all entry points of the DVD system. 