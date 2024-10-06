# newdatepicker
currently the new datepicker

<?php

$conn = new mysqli('localhost', 'root', '', 'rostermanagement') or die('Connection unsuccessful');

$sql = 'SELECT * FROM dateandtime WHERE id = "1"';
$result = $conn->query($sql);

$row = $result->fetch_assoc();

// Get the start and end dates from the database
$mondayDateFromDB = $row['startdate']; 
$saturdayDateFromDB = $row['enddate']; 

$mondayDate = new DateTime($mondayDateFromDB);
$saturdayDate = new DateTime($saturdayDateFromDB);

$currentDate = new DateTime();

// Function to calculate the next Monday and Saturday
function getNextMondaysAndSaturdays($mondayDate, $saturdayDate) {
    // Clone the original dates for manipulation
    $nextMonday = clone $mondayDate;
    $nextSaturday = clone $saturdayDate;

    // to get the next monday on the basis of current monday 
    $currentMondayexistence = $nextMonday->format('Y-m-d');

    // Get the current date for comparison
    $currentDate = new DateTime($currentMondayexistence);

    // Check if the start date is not a Monday
    if ($nextMonday->format('w') != 1) { // 'w' returns the numeric representation of the day of the week (0 for Sunday, 1 for Monday, etc.)
        // If it's not Monday, find the next Monday
        $nextMonday->modify('next  monday');
    }

    // If the next Monday is in the past or today, find the next Monday
    if ($nextMonday <= $currentDate) {
        $nextMonday->modify('next  monday');
    }

    // Calculate the corresponding Saturday from the next Monday
    $nextSaturday = clone $nextMonday;
    $nextSaturday->modify('+5 days'); // Saturday is 5 days after Monday

    return [
        'next_monday' => $nextMonday->format('Y-m-d'),
        'next_saturday' => $nextSaturday->format('Y-m-d')
    ];
}

// Get the next Monday and Saturday dates
$nextDates = getNextMondaysAndSaturdays($mondayDate, $saturdayDate);

echo json_encode($nextDates);
?>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>jQuery UI Datepicker - Default functionality</title>
    <link rel="stylesheet" href="https://code.jquery.com/ui/1.14.0/themes/base/jquery-ui.css">
    <link rel="stylesheet" href="/resources/demos/style.css">
    <script src="https://code.jquery.com/jquery-3.7.1.js"></script>
    <script src="https://code.jquery.com/ui/1.14.0/jquery-ui.js"></script>
</head>

<body>

    <p>Monday Date: <input type="text" id="datepicker"></p>
    <p>Saturday Date :<input type="text" id="saturdayfill"></p>

    <script>
        $(document).ready(function() {
            $.ajax({
                type: "GET",
                url: "read.php",
                dataType: "json",
                success: function(data) {
                    var mindate = data.next_monday;
                    var maxdate = data.next_saturday;

                    $(function() {
                        $("#datepicker").datepicker({
                            dateFormat: "yy-mm-dd",
                            minDate: mindate,
                            maxDate: maxdate,
                            beforeShowDay: function(date) {
                                var day = date.getDay();
                                return [(day === 1 || day === 6), ""];
                            },
                        onSelect: function(selectedDate) {
                            var selectedDay = new Date(selectedDate).getDay();

                            // Check if the selected day is Monday
                            if (selectedDay === 1) {
                                // Calculate the next Saturday based on the selected Monday
                                var selectedDateObj = new Date(selectedDate);
                                var nextSaturday = new Date(selectedDateObj);
                                nextSaturday.setDate(selectedDateObj.getDate() + 5); // Add 5 days to Monday to get Saturday

                                // Format the next Saturday date to 'yy-mm-dd'
                                var formattedDate = $.datepicker.formatDate("yy-mm-dd", nextSaturday);

                                // Auto-fill the Saturday date in the second datepicker field
                                $("#saturdayfill").val(formattedDate);
                            }
                        }
                    })
                })
                }
            });
        });
    </script>
    <script>
        // Event that triggers when a date is selected


        // Initialize the second datepicker (optional)
        $("#saturdayfill").datepicker({
            dateFormat: "yy-mm-dd",
        });
        //   });
    </script>

</body>

</html>

