

# Fix: Next Button Disabled on Design Preview Step

## Problem Identified

The "Next" button is disabled on Step 3 (Design Preview) because **Step 3 is never marked as complete** in the wizard flow.

## Root Cause

Looking at the navigation logic in `Analyze.tsx`:
```tsx
const canGoForward = currentStep < 5 && completedSteps.includes(currentStep as any);
```

The button is only enabled when the **current step is in the `completedSteps` array**. 

In `useAnalysis.ts`, when the analysis completes (line 135-136):
```tsx
markStepComplete(2);  // Mark step 2 done
setCurrentStep(3);     // Move to step 3
```

Step 3 is never marked as complete, so `completedSteps.includes(3)` returns `false`.

---

## Solution

Add `markStepComplete(3)` after the analysis is complete and the user lands on Step 3. Since Step 3 (Design Preview) is a viewing step with no required action, it should be marked complete immediately when the markdown design is available.

---

## Technical Changes

### File: `src/hooks/useAnalysis.ts`

**Change:** After setting the analysis result and moving to Step 3, also mark Step 3 as complete.

**Location:** Line 135-136

**Current Code:**
```tsx
setAnalysisResult(mappedResult);
markStepComplete(2);
setCurrentStep(3);
```

**Updated Code:**
```tsx
setAnalysisResult(mappedResult);
markStepComplete(2);
markStepComplete(3);  // Mark step 3 complete since it's a view-only step
setCurrentStep(3);
```

---

## Expected Result

After this fix:
1. User uploads image (Step 1)
2. Analysis runs (Step 2) 
3. When analysis completes, steps 1, 2, and 3 are marked complete
4. User lands on Step 3 (Design Preview) with the Next button **enabled**
5. User can proceed to Step 4 (Refinement)

