## API Source

API is from **data.cdc.gov** titled:

**“Cardiovascular Disease Death Rates, Trends, and Excess Death Rates Among US Adults (35+) by County and Age Group – 2010–2020”**

### API Endpoint
https://data.cdc.gov/resource/rsk5-566a.json


## Laravel Backend Setup

### CDC Data Service

```php
// app/Services/CDCDataService.php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;

class CDCDataService
{
    private $baseUrl = 'https://data.cdc.gov/resource/rsk5-566a.json';

    public function getCardiovascularData($limit = 100, $offset = 0)
    {
        $response = Http::get($this->baseUrl, [
            '$limit' => $limit,
            '$offset' => $offset,
            '$order' => 'year DESC'
        ]);

        return $response->json();
    }

    public function filterByState($state, $limit = 100)
    {
        $response = Http::get($this->baseUrl, [
            '$where' => "locationabbr = '{$state}'",
            '$limit' => $limit
        ]);

        return $response->json();
    }

    public function filterByYear($year)
    {
        $response = Http::get($this->baseUrl, [
            '$where' => "year = '{$year}'",
            '$limit' => 1000
        ]);

        return $response->json();
    }
}
```

## Controller Setup

```php
// app/Http/Controllers/CardiovascularController.php
<?php

namespace App\Http\Controllers;

use App\Services\CDCDataService;
use Illuminate\Http\Request;

class CardiovascularController extends Controller
{
    protected $cdcService;

    public function __construct(CDCDataService $cdcService)
    {
        $this->cdcService = $cdcService;
    }

    public function index(Request $request)
    {
        $limit = $request->get('limit', 100);
        $offset = $request->get('offset', 0);

        $data = $this->cdcService->getCardiovascularData($limit, $offset);

        return response()->json($data);
    }

    public function byState($state)
    {
        return response()->json(
            $this->cdcService->filterByState($state)
        );
    }

    public function byYear($year)
    {
        return response()->json(
            $this->cdcService->filterByYear($year)
        );
    }
}
```

## API Routes

```php
// routes/api.php
use App\Http\Controllers\CardiovascularController;

Route::get('/cardiovascular', [CardiovascularController::class, 'index']);
Route::get('/cardiovascular/state/{state}', [CardiovascularController::class, 'byState']);
Route::get('/cardiovascular/year/{year}', [CardiovascularController::class, 'byYear']);
```

## Vue Frontend Service

```js
// resources/js/services/cardiovascularService.js
import axios from 'axios';

const API_URL = '/api/cardiovascular';

export default {
    getAll(limit = 100, offset = 0) {
        return axios.get(API_URL, { params: { limit, offset } });
    },
    getByState(state) {
        return axios.get(`${API_URL}/state/${state}`);
    },
    getByYear(year) {
        return axios.get(`${API_URL}/year/${year}`);
    }
};
```

## Vue Component

```vue
<!-- resources/js/components/CardiovascularData.vue -->
<template>
    <div class="cardiovascular-data">
        <h2>Cardiovascular Disease Data</h2>
        ...
    </div>
</template>

<script>
import cardiovascularService from '../services/cardiovascularService';
export default {
    name: 'CardiovascularData'
};
</script>

<style scoped>
.cardiovascular-data {
    padding: 20px;
}
</style>
```

Test endpoint: `https://data.cdc.gov/resource/rsk5-566a.json?$limit=10`

