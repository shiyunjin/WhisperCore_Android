WhisperCore Android - Multi-Language Transcription Library
Documentation
Overview
This is a modified version of WhisperCore_Android that enables automatic language detection and native language transcription instead of forcing English translation. The modification allows audio in any language to be transcribed in its original language.

What Was Changed
Original Behavior

All audio was transcribed/translated to English regardless of input language
Spanish audio: "Hola, ¿cómo estás?" → "Hello, how are you?"

Modified Behavior

Audio is transcribed in its detected language
Spanish audio: "Hola, ¿cómo estás?" → "Hola, ¿cómo estás?"
Supports 90+ languages automatically


Technical Implementation
Code Modification
File: whispercore/src/main/cpp/whisper-jni.cpp
Line 398 changed:
cpp// Before
params.language = "en";

// After  
params.language = "auto";
Context (lines 392-401):
cppstruct whisper_full_params params = whisper_full_default_params(WHISPER_SAMPLING_GREEDY);
params.print_realtime = false;
params.print_progress = false;
params.print_timestamps = true;
params.print_special = false;
params.translate = false;      // Already set - prevents translation
params.language = "auto";       // Changed from "en" - enables auto-detection
params.n_threads = num_threads;
params.offset_ms = 0;
params.no_context = true;
params.single_segment = false;
Why This Works
The Whisper model has built-in language detection. By setting params.language = "auto", the model:

Analyzes the first few seconds of audio
Detects the spoken language
Transcribes in that language
Does not translate (because params.translate = false)


Building the Library
Prerequisites

Android Studio with NDK installed
CMake 3.22.1+
NDK 27.0.12077973 (or compatible version)
Git

Build Steps

Clone and modify:

bashgit clone https://github.com/YOUR_USERNAME/WhisperCore_Android.git
cd WhisperCore_Android

Make the code change (line 398 in whisper-jni.cpp)
Add Maven publishing to whispercore/build.gradle.kts:

Add to plugins section:
kotlinplugins {
  alias(libs.plugins.android.library)
  alias(libs.plugins.kotlin.android)
  id("maven-publish")  // Add this
}
Add at end of file:
kotlinafterEvaluate {
    publishing {
        publications {
            create<MavenPublication>("release") {
                from(components["release"])
                groupId = "com.redravencomputing"
                artifactId = "whispercore"
                version = "1.0.0-SNAPSHOT"
            }
        }
    }
}

Build and publish:

bash./gradlew clean
./gradlew :whispercore:assembleRelease
./gradlew :whispercore:publishToMavenLocal
Build time: ~3-5 minutes on first build

Integration into Your App
1. Configure Repositories
File: settings.gradle.kts
kotlindependencyResolutionManagement {
  repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
  repositories {
    mavenLocal()  // Must be first to use your modified version
    google()
    mavenCentral()
  }
}
2. Add Dependency
File: app/build.gradle.kts
kotlindependencies {
    implementation("com.redravencomputing:whispercore:1.0.0-SNAPSHOT")
    // ... other dependencies
}
3. Sync and Build
bash./gradlew clean
./gradlew assembleDebug

Usage
The API remains unchanged from the original WhisperCore_Android library. The only difference is the transcription output.
Basic Example
kotlin// Initialize context with model
val context = Whisper.initContextFromAsset(
    context.assets,
    "ggml-base.bin"
)

// Transcribe audio (returns transcription in detected language)
Whisper.fullTranscribe(context, numThreads, audioData)

// Get results
val segmentCount = Whisper.getTextSegmentCount(context)
for (i in 0 until segmentCount) {
    val text = Whisper.getTextSegment(context, i)
    // Text will be in original language
}

// Clean up
Whisper.freeContext(context)
Supported Languages
The Whisper model supports 90+ languages including:

Spanish, French, German, Italian, Portuguese
Mandarin, Japanese, Korean, Hindi, Arabic
Russian, Polish, Turkish, Dutch, Swedish
And many more...

Full list: https://github.com/openai/whisper#available-models-and-languages

Models
You need to provide Whisper model files. Download from:
https://huggingface.co/ggerganov/whisper.cpp/tree/main
Recommended models:

tiny.bin - Fastest, least accurate (39 MB)
base.bin - Good balance (74 MB)
small.bin - Better accuracy (244 MB)
medium.bin - High accuracy (769 MB)
large-v3.bin - Best accuracy (1.5 GB)

Place models in assets/ folder.

Performance Considerations
Language Detection Overhead
Auto-detection adds ~50-100ms to initial processing but happens only once per audio file.
Model Size vs Speed
ModelSizeSpeedAccuracytiny39 MBFastestBasicbase74 MBFastGoodsmall244 MBMediumBettermedium769 MBSlowHighlarge-v31.5 GBSlowestBest
Threading
Use numThreads = 4 for most devices. Adjust based on device CPU cores.

Troubleshooting
Library Not Found
Could not find com.redravencomputing:whispercore:1.0.0-SNAPSHOT
Solution: Ensure mavenLocal() is in dependencyResolutionManagement repositories (not just pluginManagement).
Build Fails with NDK Error
NDK not found
Solution: Install NDK via Android Studio SDK Manager → SDK Tools → NDK (Side by side)
Language Still Translating to English
Verify the code change was made and the library was rebuilt. Check the published library location:
C:\Users\USERNAME\.m2\repository\com\redravencomputing\whispercore\1.0.0-SNAPSHOT\

Comparison with Original
FeatureOriginalModifiedLanguageEnglish onlyAuto-detect 90+ languagesTranslationYesNo (transcribe only)PerformanceSameSameAPIUnchangedUnchangedModel filesSameSame

Limitations

Language mixing: If audio switches languages mid-stream, accuracy may decrease
Accents: Strong accents in any language may affect accuracy
Background noise: Affects all languages equally
Code-switching: Switching between languages in same utterance not well supported


License
This modified version inherits the license from the original WhisperCore_Android repository. Whisper itself is MIT licensed by OpenAI.

Credits

Original library: EberronBruce/WhisperCore_Android
Whisper model: OpenAI
whisper.cpp: Georgi Gerganov


Future Improvements
Potential enhancements:

Add language-specific parameters (e.g., better punctuation for specific languages)
Expose language detection confidence scores
Allow manual language override
Support for language hints to improve detection accuracy


Support
For issues with:

This modification: Check the C++ code change was applied correctly
Original library: Refer to original WhisperCore_Android repository
Whisper model: Refer to whisper.cpp documentation
