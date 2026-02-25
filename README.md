<?php
require_once __DIR__ . '/../../../config/config.php';
require_once __DIR__ . '/../../../helpers/DateHelper.php';

header('Content-Type: application/json');

function debugLog($msg) {
    file_put_contents(__DIR__ . '/debug_get_weeks.log', date('Y-m-d H:i:s') . ' - ' . $msg . PHP_EOL, FILE_APPEND);
}

debugLog("1. Started get_weeks.php");

// Check authentication
if (!isLoggedIn()) {
    debugLog("Auth failed");
    http_response_code(401);
    echo json_encode(['success' => false, 'message' => 'Unauthorized']);
    exit;
}

try {
    debugLog("2. Instantiating DB");
    $database = new MyDatabase();
    
    debugLog("3. Connecting DB");
    $conn = $database->connect_db();
    
    debugLog("4. Instantiating DateHelper");
    $dateHelper = new DateHelper($conn);

    debugLog("5. Querying min/max dates");
    // Get date range from log_data
    $query = "SELECT 
                MIN(tanggal_pengajuan) as min_date,
                MAX(tanggal_pengajuan) as max_date
              FROM dbo.invenlist_log_data
              WHERE tanggal_pengajuan IS NOT NULL";

    $stmt = sqlsrv_query($conn, $query);

    if (!$stmt) {
        debugLog("6. Query failed: " . print_r(sqlsrv_errors(), true));
        throw new Exception("Failed to fetch date range");
    }

    $result = sqlsrv_fetch_array($stmt, SQLSRV_FETCH_ASSOC);

    if (!$result || !$result['min_date']) {
        debugLog("7. No data available");
        // No data available
        echo json_encode([
            'success' => true,
            'weeks'   => []
        ]);
        exit;
    }

    $minDate = $result['min_date'];
    $maxDate = $result['max_date'];

    debugLog("8. Min date: " . (is_object($minDate) ? $minDate->format('Y-m-d') : $minDate));
    
    // Convert to DateTime objects
    $startDate = ($minDate instanceof DateTime) ? $minDate : new DateTime($minDate);
    $endDate   = ($maxDate instanceof DateTime) ? $maxDate : new DateTime($maxDate);

    // Find the first Monday on or before startDate
    $current = clone $startDate;
    while ($current->format('N') != 1) { // 1 = Monday
        $current->modify('-1 day');
    }

    $weeks      = [];
    $weekNumber = 1;
    $previousCutoff = null;

    debugLog("9. Starting loop");
    // Loop until we cover the max date
    while ($previousCutoff === null || $previousCutoff < $endDate) {
        // 1. Determine the nominal start of the week (Monday)
        $nominalStart = clone $current;

        // 2. Calculate the cutoff date (Friday or earlier if holiday)
        $cutoffDate = $dateHelper->calculateCutoffDate($nominalStart);

        // 3. Set the cutoff time to 12:00:00
        $weekEndTimestamp = clone $cutoffDate;
        $weekEndTimestamp->setTime(12, 0, 0);

        // 4. Determine the start timestamp for this week
        if ($previousCutoff === null) {
            // First week: Start from Monday 00:00:00
            $weekStartTimestamp = clone $nominalStart;
            $weekStartTimestamp->setTime(0, 0, 0);
        } else {
            // Subsequent weeks: Start immediately after previous cutoff
            $weekStartTimestamp = clone $previousCutoff;
        }

        // 5. Count records in this week range
        $operator   = ($previousCutoff === null) ? '>=' : '>';
        $countQuery = "SELECT COUNT(*) as count 
                       FROM dbo.invenlist_log_data 
                       WHERE tanggal_pengajuan $operator ? 
                       AND tanggal_pengajuan <= ?";

        $countParams = [
            $weekStartTimestamp->format('Y-m-d H:i:s'),
            $weekEndTimestamp->format('Y-m-d H:i:s')
        ];

        $countStmt = sqlsrv_query($conn, $countQuery, $countParams);
        if ($countStmt === false) {
            $count = 0;
        } else {
            $countResult = sqlsrv_fetch_array($countStmt, SQLSRV_FETCH_ASSOC);
            $count = $countResult['count'] ?? 0;
        }

        // 6. Add week to list (only if it has data)
        if ($count > 0) {
            $weeks[] = [
                'week_number' => $weekNumber,
                'start_date'  => $weekStartTimestamp->format('Y-m-d H:i:s'),
                'end_date'    => $weekEndTimestamp->format('Y-m-d H:i:s'),
                'label'       => sprintf(
                    'Minggu %d (%s - %s)',
                    $weekNumber,
                    $nominalStart->format('d M'),
                    $cutoffDate->format('d M Y')
                ),
                'count' => $count
            ];
        }

        // 7. Update previous cutoff and advance to next Monday
        $previousCutoff = clone $weekEndTimestamp;
        $current->modify('+7 days');
        $weekNumber++;
    }

    debugLog("10. Loop finished, returning JSON");
    $database->disconnect();

    echo json_encode([
        'success' => true,
        'weeks'   => $weeks
    ]);

} catch (Exception $e) {
    debugLog("Exception caught: " . $e->getMessage());
    http_response_code(500);
    echo json_encode([
        'success' => false,
        'message' => $e->getMessage()
    ]);
} catch (Error $e) {
    debugLog("Fatal Error caught: " . $e->getMessage() . " at " . $e->getFile() . ":" . $e->getLine());
    http_response_code(500);
    echo json_encode([
        'success' => false,
        'message' => "Internal Server Error"
    ]);
}
?>
