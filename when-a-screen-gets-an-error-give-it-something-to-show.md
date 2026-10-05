# When a screen gets an error, give it something to show

A screen can handle loading and success correctly and still crash the first time its data source fails. A May 2026 issue in Google's Now in Android sample described exactly that shape of bug: an error state reached `TODO()` in a screen and threw at runtime. The useful lesson applies to any app: if a state can reach production, the UI needs a deliberate result for it.

Google's UI architecture guidance describes UI state as the information the screen needs to render, with a state holder producing that state and the UI displaying it. That gives errors a clear path: map the failure to a UI state, show a useful recovery option, and test that branch. [Android UI layer](https://developer.android.com/topic/architecture/ui-layer) · [Now in Android issue #2108](https://github.com/android/nowinandroid/issues/2108)

## Model the states the screen can receive

For example, a detail screen could use a small sealed state:

```kotlin
sealed interface TopicUiState {
    data object Loading : TopicUiState
    data class Content(val title: String) : TopicUiState
    data object Error : TopicUiState
}
```

Then render every case:

```kotlin
@Composable
fun TopicScreen(
    state: TopicUiState,
    onRetry: () -> Unit,
) {
    when (state) {
        TopicUiState.Loading -> LoadingContent()
        is TopicUiState.Content -> TopicContent(title = state.title)
        TopicUiState.Error -> ErrorContent(onRetry = onRetry)
    }
}
```

ErrorContent can explain that the screen could not load and offer Retry or Back. Keep exception details in diagnostics rather than showing raw messages to users. If the screen can keep useful old data after a refresh fails, model that explicitly—for example, content plus a non-blocking error—so an error does not unnecessarily erase the last good view.

## Exercise the failure path

1. Make the repository return an error in a ViewModel or state-holder test.
2. Check that it emits the expected error state and that retry starts the load again.
3. In a UI test, provide that state and verify the error message and recovery action appear.
4. Try the same state with an empty list or missing item if those are separate cases in your screen.

The point is not to test every exception type in the UI. Test the states the UI promises to handle. An exhaustive `when` over a sealed state also helps the compiler point out a newly added state that has no presentation.

In the Now in Android report, the failing branch was not covered by the screen test, which exercised Loading and Success only. Treat it as a useful reminder to include error states in the UI test matrix. The report is one example; it does not say how often this happens across Android apps.

When a repository or network call fails, the user should still see a screen with a clear next step. Make that state part of the design, then make it part of the test.
