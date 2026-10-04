# Interviewer-Only Evaluation Guide

**Do not include this directory in the candidate copy.**

## Expected backend approach

Validate:

- `change` is a non-zero integer
- `reason` is a non-empty string after trimming

A strong implementation uses a conditional atomic update:

```js
const change = Number(req.body.change)
const reason = req.body.reason?.trim()

if (!Number.isInteger(change) || change === 0 || !reason) {
  return res.status(400).send({
    message: "A non-zero integer change and reason are required",
  })
}

const filter = {
  sku: req.params.sku,
  deleted: { $ne: true },
  ...(change < 0 ? { stock: { $gte: Math.abs(change) } } : {}),
}

const product = await Product.findOneAndUpdate(
  filter,
  {
    $inc: { stock: change },
    $push: {
      stockAdjustments: {
        change,
        reason,
        adjustedAt: new Date(),
      },
    },
  },
  { new: true, runValidators: true }
)

if (!product) {
  const exists = await Product.exists({
    sku: req.params.sku,
    deleted: { $ne: true },
  })

  if (!exists) {
    return res.status(404).send({ message: "Product not found" })
  }

  return res.status(409).send({ message: "Insufficient stock" })
}

res.send(product)
```

A read-check-save implementation can still pass if it is correct and the
candidate clearly identifies its race condition during discussion.

## Expected Redux action

```js
dispatch({ type: "SET_PRODUCT_LOADING", data: true })
dispatch({ type: "SET_PRODUCT_ERROR", data: "" })

try {
  const response = await axios.patch(
    `${apiUrl}/product/${sku}/stock`,
    { change, reason }
  )

  dispatch({
    type: "ADJUST_STOCK_SUCCESS",
    data: response.data,
  })

  return true
} catch (error) {
  dispatch({
    type: "SET_PRODUCT_ERROR",
    data:
      error.response?.data?.message || "Could not adjust product stock",
  })
  return false
} finally {
  dispatch({ type: "SET_PRODUCT_LOADING", data: false })
}
```

## Manual acceptance checks

1. Use **Add stock** with quantity `5` and a reason: stock increases and UI updates.
2. Use **Remove stock** with quantity `2`: stock decreases and UI updates.
3. Remove more than available: API returns `409`; stock is unchanged.
4. Submit `0`, a decimal or blank reason: API returns `400`.
5. Use an unknown SKU: API returns `404`.
6. Confirm a successful update appears in `stockAdjustments`.

## Scoring — 30

| Category | Points |
| --- | ---: |
| Business rules and validation | 8 |
| Express endpoint and status codes | 6 |
| MongoDB update logic | 5 |
| Redux/Axios integration | 5 |
| Loading and error handling | 3 |
| Code quality and explanation | 3 |

- 25–30: Strong
- 21–24: Pass
- 16–20: Consider with concerns
- Below 16: Do not proceed
