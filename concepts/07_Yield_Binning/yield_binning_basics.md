# Yield and Binning Basics

## 1. What is Yield?

* **Yield** is one of the most critical metrics in semiconductor manufacturing. It represents the percentage of manufactured devices that successfully pass all required tests and meet specifications.
* **Calculation:**
    ```
    Yield (%) = (Number of Good Devices / Total Number of Devices Tested) * 100%
    ```
* **Types of Yield:**
    * **Wafer Sort Yield (or Die Yield):** Percentage of good dies found on a wafer before packaging.
    * **Final Test Yield (or Package Yield):** Percentage of packaged devices that pass final testing.
    * **Overall Yield:** Combined yield across multiple stages (Sort Yield * Test Yield * other steps).
* **Importance:** Yield directly impacts the cost per chip and the profitability of manufacturing. Low yield indicates problems in the design, manufacturing process, or testing itself. Continuous yield improvement is a primary goal.

## 2. What is Binning?

* **Binning** is the process of categorizing or sorting tested devices into different groups ("bins") based on the results obtained from the ATE test program.
* Each bin typically corresponds to a specific pass/fail outcome or performance characteristic.
* Think of it like sorting mail into different slots based on the address or delivery instructions.

## 3. Purpose of Binning

* **Separating Good from Bad:** The most basic purpose is to identify devices that pass all tests (e.g., "Bin 1") and separate them from devices that fail *any* test.
* **Failure Diagnosis:** Assigning failing devices to different bins based on *which* test(s) they failed provides valuable diagnostic information. Examples:
    * A bin for devices failing basic continuity/shorts tests (indicates assembly/package issues).
    * A bin for devices drawing excessive power (indicates power grid shorts or defects).
    * Bins for failures in specific functional blocks (identified by ATPG patterns or functional tests).
    * Bins for memory failures (identified by BIST results).
    * This failure data helps engineers pinpoint problems in the design or manufacturing process.
* **Performance Grading:** For some products, binning can be used to sort passing devices into different performance grades (e.g., faster vs. slower speed grades based on timing tests). These different grades can sometimes be sold at different price points.
* **Quality Control:** Tracking the number of devices falling into each bin over time helps monitor the stability and health of the manufacturing process.

## 4. Example Bin Categories

* Bin numbers and definitions are specific to each device and company, but common categories include:
    * **Bin 1:** Pass - Device passed all tests. (The "money bin").
    * **Bin 2:** Contact/Continuity Fail - ATE couldn't make good electrical contact.
    * **Bin 3:** Gross Power/Shorts Fail - Device failed basic power supply checks.
    * **Bin 4-X:** Functional Failures - Device failed specific logic, timing, or analog tests. Might be broken down by test section.
    * **Bin Y:** Memory BIST Fail - Device failed the on-chip memory test.
    * **Bin Z:** Parametric Fail - Device failed tests related to voltage/current levels, timing margins, etc.
    * **Bin 255 (or similar high number):** Often reserved for devices that couldn't be properly tested due to setup issues or errors.

## 5. Relevance

Yield and Binning connect the detailed electrical testing (like ATPG results, BIST pass/fail flags, JTAG tests, functional results obtained on **ATE**) directly to the manufacturing and business outcomes. Analyzing bin distributions is key to improving yield and product quality.
