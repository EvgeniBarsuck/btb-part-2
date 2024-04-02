Task: Please write a stored procedure that creates an amortization schedule (the Spitzer
Table) for a loan of 36000 and interest of 8% for 36 monthly payments.
After the 12th payment, you are required to change the loan’s setup - perform a loan
recycle on the remaining amount (according to the amortization schedule table) with
a fixed interest of 4.5% for an additional 48 payments.

Solution:

    CREATE PROCEDURE GenerateAmortizationSchedule AS
    BEGIN
    SET NOCOUNT ON;

    -- Declarations
    DECLARE @initialLoanAmount MONEY = 36000, @initialRate DECIMAL(5, 2) = 8.0,
            @initialTerm INT = 36, @recycleRate DECIMAL(5, 2) = 4.5, @recycleTerm INT = 48,
            @monthlyPayment MONEY, @balance MONEY, @monthlyRate DECIMAL(10, 7),
            @recycleMonthlyPayment MONEY, @interestPayment MONEY, @principalPayment MONEY,
            @paymentNumber INT = 1, @totalPayments INT;

    -- Temporary table to hold the amortization schedule
    CREATE TABLE #AmortizationSchedule (
        PaymentNumber INT,
        PaymentAmount MONEY,
        PrincipalPayment MONEY,
        InterestPayment MONEY,
        RemainingBalance MONEY
    );

    -- Initial monthly rate
    SET @monthlyRate = @initialRate / 100 / 12;

    -- Calculate initial monthly payment (using approximation for PMT formula)
    SET @monthlyPayment = @initialLoanAmount * (@monthlyRate / (1 - POWER(1 + @monthlyRate, -@initialTerm)));

    -- Set initial balance to the loan amount
    SET @balance = @initialLoanAmount;

    -- Generate schedule for the first 12 payments
    WHILE @paymentNumber <= 12
    BEGIN
        SET @interestPayment = @balance * @monthlyRate;
        SET @principalPayment = @monthlyPayment - @interestPayment;
        SET @balance = @balance - @principalPayment;

        INSERT INTO #AmortizationSchedule (PaymentNumber, PaymentAmount, PrincipalPayment, InterestPayment, RemainingBalance)
        VALUES (@paymentNumber, @monthlyPayment, @principalPayment, @interestPayment, @balance);

        SET @paymentNumber = @paymentNumber + 1;
    END

    -- Calculate the remaining balance and set up for loan recycle
    -- No need to recalculate @balance, as it's already set

    -- Recalculate monthly payment for the recycled loan
    SET @monthlyRate = @recycleRate / 100 / 12;
    SET @recycleMonthlyPayment = @balance * (@monthlyRate / (1 - POWER(1 + @monthlyRate, -@recycleTerm)));

    -- Generate schedule for the next 48 payments
    SET @totalPayments = @paymentNumber + @recycleTerm - 1; -- Adjust for continuing payment numbers
    WHILE @paymentNumber <= @totalPayments
    BEGIN
        SET @interestPayment = @balance * @monthlyRate;
        SET @principalPayment = @recycleMonthlyPayment - @interestPayment;
        SET @balance = @balance - @principalPayment;

        INSERT INTO #AmortizationSchedule (PaymentNumber, PaymentAmount, PrincipalPayment, InterestPayment, RemainingBalance)
        VALUES (@paymentNumber, @recycleMonthlyPayment, @principalPayment, @interestPayment, @balance);

        SET @paymentNumber = @paymentNumber + 1;
    END

    -- Select the amortization schedule to return the result
    SELECT * FROM #AmortizationSchedule;

    -- Clean up
    DROP TABLE #AmortizationSchedule;
END
GO
