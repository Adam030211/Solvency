import SwiftUI

// MARK: - Models (unchanged from your code)
enum CompoundingFrequency: Int, CaseIterable, Identifiable {
    case monthly = 12
    case quarterly = 4
    case semiAnnually = 2
    case annually = 1
    
    var id: Int { self.rawValue }
    
    var description: String {
        switch self {
        case .monthly: return "Monthly"
        case .quarterly: return "Quarterly"
        case .semiAnnually: return "Semi-Annually"
        case .annually: return "Annually"
        }
    }
}

enum LoanType: String, CaseIterable, Identifiable {
    case annuity = "Annuity Loan"
    case straight = "Straight Loan"
    case bullet = "Bullet Loan"
    case adjustableRate = "Adjustable-Rate Loan"
    case fixedRate = "Fixed-Rate Loan"
    case balloon = "Balloon Loan"
    case interestOnly = "Interest-Only Loan"
    
    var id: String { self.rawValue }
    
    var description: String {
        switch self {
        case .annuity: return "Fixed periodic payments, with interest decreasing and principal increasing over time"
        case .straight: return "Fixed principal payments, with decreasing total payments over time"
        case .bullet: return "Only interest is paid periodically; the full principal is repaid at the end"
        case .adjustableRate: return "Interest rate changes periodically, affecting payment amounts"
        case .fixedRate: return "Interest rate stays the same for a set period, keeping payments constant"
        case .balloon: return "Small payments initially, followed by a large lump-sum payment at the end"
        case .interestOnly: return "Only interest is paid for an initial period before principal payments begin"
        }
    }
}

struct LoanParameters {
    let principal: Double
    let annualInterestRate: Double
    let termInYears: Double
    let adjustmentPeriod: Double?
    let rateAdjustment: Double?
    let balloonPaymentYear: Double?
    let interestOnlyPeriod: Double?
}

struct Loan: Identifiable {
    var id = UUID()
    var loanAmount: String = "100000"
    var loanInterestRate: String = "4.5"
    var loanTerm: String = "30"
    var monthlyPayment: String = "500"
    var selectedLoanType: LoanType = .annuity
    var name: String = "Loan"
    var adjustmentPeriodYears: String = "1.0"
    var rateAdjustmentPercent: String = "0.5"
    var balloonPaymentPercentage: String = "80"
    var interestOnlyPercentage: String = "50"
}

struct Investment: Identifiable {
    var id = UUID()
    var investmentAmount: String = "100000"
    var investmentTerm: String = "30"
    var investmentReturn: String = "7.0"
    var taxRate: String = "25"
    var monthlyContribution: String = "500"
    var compoundingFrequency: CompoundingFrequency = .monthly
    var name: String = "Investment"
}

// MARK: - View Model (unchanged from your code)
class CalculatorViewModel: ObservableObject {
    @Published var loans: [Loan] = [Loan()]
    @Published var investments: [Investment] = [Investment()]
    
    func addLoan() { loans.append(Loan(name: "Loan \(loans.count + 1)")) }
    func removeLoan(at index: Int) { guard loans.count > 1, index < loans.count else { return }; loans.remove(at: index) }
    func addInvestment() { investments.append(Investment(name: "Investment \(investments.count + 1)")) }
    func removeInvestment(at index: Int) { guard investments.count > 1, index < investments.count else { return }; investments.remove(at: index) }
    
    func calculateComparison() -> (loanCost: Double, loanTotalInterest: Double, investmentValue: Double, investmentProfit: Double, netProfitLoss: Double) {
        var totalLoanCost: Double = 0
        var totalLoanInterest: Double = 0
        
        for loan in loans {
            let params = createLoanParameters(from: loan)
            let (cost, interest) = calculateLoanCost(loanType: loan.selectedLoanType, params: params)
            totalLoanCost += cost
            totalLoanInterest += interest
        }
        
        var totalInvestmentValue: Double = 0
        var totalInvestmentProfit: Double = 0
        
        for investment in investments {
            let value = calculateInvestmentValue(for: investment)
            totalInvestmentValue += value
            let principal = Double(investment.investmentAmount) ?? 0
            let monthlyContribution = Double(investment.monthlyContribution) ?? 0
            let termInYears = Double(investment.investmentTerm) ?? 0
            let totalContributions = principal + (monthlyContribution * termInYears * 12)
            totalInvestmentProfit += (value - totalContributions)
        }
        
        let netProfitLoss = totalInvestmentProfit - totalLoanInterest
        return (totalLoanCost, totalLoanInterest, totalInvestmentValue, totalInvestmentProfit, netProfitLoss)
    }
    
    func calculateMonthlyPayment(for loan: Loan) -> Double {
        let params = createLoanParameters(from: loan)
        return calculateMonthlyPayment(loanType: loan.selectedLoanType, params: params)
    }
    
    func createLoanParameters(from loan: Loan) -> LoanParameters {
        let principal = Double(loan.loanAmount) ?? 0
        let rate = (Double(loan.loanInterestRate) ?? 0) / 100.0
        let term = Double(loan.loanTerm) ?? 0
        
        var adjustmentPeriod: Double? = nil
        var rateAdjustment: Double? = nil
        var balloonPaymentYear: Double? = nil
        var interestOnlyPeriod: Double? = nil
        
        switch loan.selectedLoanType {
        case .adjustableRate:
            adjustmentPeriod = Double(loan.adjustmentPeriodYears) ?? 1.0
            rateAdjustment = (Double(loan.rateAdjustmentPercent) ?? 0.5) / 100.0
        case .balloon:
            let percentage = Double(loan.balloonPaymentPercentage) ?? 80.0
            balloonPaymentYear = term * (percentage / 100.0)
        case .interestOnly:
            let percentage = Double(loan.interestOnlyPercentage) ?? 50.0
            interestOnlyPeriod = term * (percentage / 100.0)
        default: break
        }
        
        return LoanParameters(
            principal: principal,
            annualInterestRate: rate,
            termInYears: term,
            adjustmentPeriod: adjustmentPeriod,
            rateAdjustment: rateAdjustment,
            balloonPaymentYear: balloonPaymentYear,
            interestOnlyPeriod: interestOnlyPeriod
        )
    }
    
    func calculateInvestmentValue(for investment: Investment) -> Double {
        let principal = Double(investment.investmentAmount) ?? 0
        let annualRate = (Double(investment.investmentReturn) ?? 0) / 100.0
        let tax = Double(investment.taxRate) ?? 0
        let term = Double(investment.investmentTerm) ?? 0
        let monthlyContribution = Double(investment.monthlyContribution) ?? 0
        let periodsPerYear = Double(investment.compoundingFrequency.rawValue)
        let totalPeriods = term * periodsPerYear
        let ratePerPeriod = pow(1 + annualRate, 1.0 / periodsPerYear) - 1.0
        let principalFV = principal * pow(1 + ratePerPeriod, totalPeriods)
        
        let contributionsPerPeriod = 12.0 / periodsPerYear
        let contributionAmount = monthlyContribution * contributionsPerPeriod
        
        var contributionsFV = 0.0
        if ratePerPeriod > 0 {
            contributionsFV = contributionAmount * (pow(1 + ratePerPeriod, totalPeriods) - 1) / ratePerPeriod
        } else {
            contributionsFV = contributionAmount * totalPeriods
        }
        
        let totalFV = principalFV + contributionsFV
        let totalContributions = principal + (monthlyContribution * term * 12)
        let taxableGains = totalFV - totalContributions
        let taxPaid = taxableGains * (tax / 100)
        
        return totalFV - taxPaid
    }
    
    func calculateMonthlyPayment(loanType: LoanType, params: LoanParameters) -> Double {
        let principal = params.principal
        let monthlyRate = params.annualInterestRate / 12.0
        let totalMonths = params.termInYears * 12.0
        
        switch loanType {
        case .annuity, .fixedRate:
            if monthlyRate == 0 { return principal / totalMonths }
            let factor = pow(1 + monthlyRate, totalMonths)
            return principal * monthlyRate * factor / (factor - 1)
        case .straight:
            let principalPayment = principal / totalMonths
            return principalPayment + (principal * monthlyRate)
        case .bullet, .interestOnly:
            return principal * monthlyRate
        case .adjustableRate:
            if monthlyRate == 0 { return principal / totalMonths }
            let factor = pow(1 + monthlyRate, totalMonths)
            return principal * monthlyRate * factor / (factor - 1)
        case .balloon:
            if monthlyRate == 0 { return principal / totalMonths }
            let factor = pow(1 + monthlyRate, totalMonths)
            return principal * monthlyRate * factor / (factor - 1)
        }
    }
    
    func calculateLoanCost(loanType: LoanType, params: LoanParameters) -> (totalCost: Double, interestPaid: Double) {
        let principal = params.principal
        let monthlyRate = params.annualInterestRate / 12.0
        let totalMonths = params.termInYears * 12.0
        var totalPayments = 0.0
        var remainingPrincipal = principal
        
        switch loanType {
        case .annuity, .fixedRate:
            let monthlyPayment = calculateMonthlyPayment(loanType: .annuity, params: params)
            totalPayments = monthlyPayment * totalMonths
        case .straight:
            let principalPayment = principal / totalMonths
            for _ in 1...Int(totalMonths) {
                let interestPayment = remainingPrincipal * monthlyRate
                totalPayments += (principalPayment + interestPayment)
                remainingPrincipal -= principalPayment
            }
        case .bullet:
            totalPayments = (principal * monthlyRate * totalMonths) + principal
        case .adjustableRate:
            let adjustmentPeriod = params.adjustmentPeriod ?? 1.0
            let rateAdjustment = params.rateAdjustment ?? 0.0
            var currentRate = monthlyRate
            var currentMonths = 0
            
            while currentMonths < Int(totalMonths) {
                let adjustmentMonths = Int(adjustmentPeriod * 12.0)
                let monthsToCalculate = min(adjustmentMonths, Int(totalMonths) - currentMonths)
                
                let tempParams = LoanParameters(
                    principal: remainingPrincipal,
                    annualInterestRate: currentRate * 12.0,
                    termInYears: Double(monthsToCalculate) / 12.0,
                    adjustmentPeriod: nil,
                    rateAdjustment: nil,
                    balloonPaymentYear: nil,
                    interestOnlyPeriod: nil
                )
                
                let periodPayment = calculateMonthlyPayment(loanType: .annuity, params: tempParams)
                
                for _ in 1...monthsToCalculate {
                    let interestPayment = remainingPrincipal * currentRate
                    let principalPayment = periodPayment - interestPayment
                    remainingPrincipal -= principalPayment
                    totalPayments += periodPayment
                }
                
                currentRate += (rateAdjustment / 12.0) / adjustmentPeriod
                currentMonths += monthsToCalculate
            }
        case .balloon:
            let balloonMonth = Int((params.balloonPaymentYear ?? params.termInYears) * 12.0)
            let monthlyPayment = calculateMonthlyPayment(loanType: .balloon, params: params)
            
            for _ in 1...balloonMonth {
                let interestPayment = remainingPrincipal * monthlyRate
                let principalPayment = min(monthlyPayment - interestPayment, remainingPrincipal)
                remainingPrincipal -= principalPayment
                totalPayments += (principalPayment + interestPayment)
            }
            
            totalPayments += remainingPrincipal
        case .interestOnly:
            let interestOnlyMonths = Int((params.interestOnlyPeriod ?? 0.0) * 12.0)
            let remainingMonths = Int(totalMonths) - interestOnlyMonths
            
            for _ in 1...interestOnlyMonths {
                let interestPayment = remainingPrincipal * monthlyRate
                totalPayments += interestPayment
            }
            
            if remainingMonths > 0 {
                let remainingTerm = Double(remainingMonths) / 12.0
                let tempParams = LoanParameters(
                    principal: remainingPrincipal,
                    annualInterestRate: params.annualInterestRate,
                    termInYears: remainingTerm,
                    adjustmentPeriod: nil,
                    rateAdjustment: nil,
                    balloonPaymentYear: nil,
                    interestOnlyPeriod: nil
                )
                
                let amortizedPayment = calculateMonthlyPayment(loanType: .annuity, params: tempParams)
                
                for _ in 1...remainingMonths {
                    let interestPayment = remainingPrincipal * monthlyRate
                    let principalPayment = min(amortizedPayment - interestPayment, remainingPrincipal)
                    remainingPrincipal -= principalPayment
                    totalPayments += (principalPayment + interestPayment)
                }
            }
        }
        
        let interestPaid = totalPayments - principal
        return (totalPayments, interestPaid)
    }
}

// MARK: - Views (updated with proper focus management)
struct LoanView: View {
    @Binding var loan: Loan
    @ObservedObject var viewModel: CalculatorViewModel
    let index: Int
    @FocusState private var focusedField: Field?
    
    enum Field: Hashable {
        case name, amount, rate, term, adjPeriod, rateAdj, balloon, interestOnly, overridePayment
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                TextField("Name", text: $loan.name)
                    .font(.headline)
                    .focused($focusedField, equals: .name)
                
                Spacer()
                
                Button(action: {
                    withAnimation {
                        viewModel.removeLoan(at: index)
                    }
                }) {
                    Image(systemName: "trash")
                        .foregroundColor(.red)
                }
                .disabled(viewModel.loans.count <= 1)
            }
            
            TextField("Loan Amount", text: $loan.loanAmount)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .amount)
                
            TextField("Interest Rate (%)", text: $loan.loanInterestRate)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .rate)
                
            TextField("Loan Term (years)", text: $loan.loanTerm)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .term)
            
            Picker("Loan Type", selection: $loan.selectedLoanType) {
                ForEach(LoanType.allCases) { loanType in
                    Text(loanType.rawValue).tag(loanType)
                }
            }
            .pickerStyle(MenuPickerStyle())
            
            Text(loan.selectedLoanType.description)
                .font(.caption)
                .foregroundColor(.secondary)
            
            if loan.selectedLoanType == .adjustableRate {
                TextField("Adjustment Period (years)", text: $loan.adjustmentPeriodYears)
                    .keyboardType(.numbersAndPunctuation)
                    .focused($focusedField, equals: .adjPeriod)
                    
                TextField("Rate Adjustment (%)", text: $loan.rateAdjustmentPercent)
                    .keyboardType(.numbersAndPunctuation)
                    .focused($focusedField, equals: .rateAdj)
            } else if loan.selectedLoanType == .balloon {
                TextField("Balloon Payment at % of Term", text: $loan.balloonPaymentPercentage)
                    .keyboardType(.numbersAndPunctuation)
                    .focused($focusedField, equals: .balloon)
            } else if loan.selectedLoanType == .interestOnly {
                TextField("Interest-Only Period (% of Term)", text: $loan.interestOnlyPercentage)
                    .keyboardType(.numbersAndPunctuation)
                    .focused($focusedField, equals: .interestOnly)
            }
            
            let payment = viewModel.calculateMonthlyPayment(for: loan)
            Text("Monthly Payment: $\(String(format: "%.2f", payment))")
                .font(.headline)
            
            TextField("Override Monthly Payment", text: $loan.monthlyPayment)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .overridePayment)
        }
        .toolbar {
            ToolbarItemGroup(placement: .keyboard) {
                            if focusedField != nil {  // Only show when this view's fields are focused
                                Spacer()
                                Button("Done") {
                                    focusedField = nil
                                }
                }
            }
        }
    }
}

struct InvestmentView: View {
    @Binding var investment: Investment
    @ObservedObject var viewModel: CalculatorViewModel
    let index: Int
    @FocusState private var focusedField: Field?
    
    enum Field: Hashable {
        case name, amount, term, returnRate, contribution, tax
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                TextField("Name", text: $investment.name)
                    .font(.headline)
                    .focused($focusedField, equals: .name)
                
                Spacer()
                
                Button(action: {
                    withAnimation {
                        viewModel.removeInvestment(at: index)
                    }
                }) {
                    Image(systemName: "trash")
                        .foregroundColor(.red)
                }
                .disabled(viewModel.investments.count <= 1)
            }
            
            TextField("Investment Amount", text: $investment.investmentAmount)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .amount)
                
            TextField("Investment Term (years)", text: $investment.investmentTerm)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .term)
                
            TextField("Expected Return (%)", text: $investment.investmentReturn)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .returnRate)
                
            TextField("Monthly Contribution", text: $investment.monthlyContribution)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .contribution)
                
            TextField("Tax Rate (%)", text: $investment.taxRate)
                .keyboardType(.numbersAndPunctuation)
                .focused($focusedField, equals: .tax)
            
            Picker("Compounding Frequency", selection: $investment.compoundingFrequency) {
                ForEach(CompoundingFrequency.allCases) { frequency in
                    Text(frequency.description).tag(frequency)
                }
            }
            .pickerStyle(MenuPickerStyle())
        }
        .toolbar {
            ToolbarItemGroup(placement: .keyboard) {
                            if focusedField != nil {  // Only show when this view's fields are focused
                                Spacer()
                                Button("Done") {
                                    focusedField = nil
                                }
                }
            }
        }
    }
}

struct ContentView: View {
    @StateObject private var viewModel = CalculatorViewModel()
    @State private var showingResults = false
    @FocusState private var isAnyFieldFocused: Bool
    
    var body: some View {
        NavigationView {
            ScrollView {
                VStack(spacing: 20) {
                    GroupBox(label: Text("Loans").font(.headline)) {
                        VStack(spacing: 15) {
                            ForEach(viewModel.loans.indices, id: \.self) { index in
                                LoanView(loan: $viewModel.loans[index], viewModel: viewModel, index: index)
                                    .padding(.vertical, 8)
                                
                                if index < viewModel.loans.count - 1 {
                                    Divider()
                                }
                            }
                            
                            Button(action: {
                                viewModel.addLoan()
                                isAnyFieldFocused = false
                            }) {
                                HStack {
                                    Image(systemName: "plus.circle.fill")
                                    Text("Add Loan")
                                }
                            }
                        }
                    }
                    .padding(.horizontal)
                    
                    GroupBox(label: Text("Investments").font(.headline)) {
                        VStack(spacing: 15) {
                            ForEach(viewModel.investments.indices, id: \.self) { index in
                                InvestmentView(investment: $viewModel.investments[index], viewModel: viewModel, index: index)
                                    .padding(.vertical, 8)
                                
                                if index < viewModel.investments.count - 1 {
                                    Divider()
                                }
                            }
                            
                            Button(action: {
                                viewModel.addInvestment()
                                isAnyFieldFocused = false
                            }) {
                                HStack {
                                    Image(systemName: "plus.circle.fill")
                                    Text("Add Investment")
                                }
                            }
                        }
                    }
                    .padding(.horizontal)
                    
                    Button("Calculate") {
                        isAnyFieldFocused = false
                        showingResults = true
                    }
                    .buttonStyle(.borderedProminent)
                    .padding(.top)
                }
                .padding(.vertical)
            }
            .navigationTitle("Loan vs Investment")
            .sheet(isPresented: $showingResults) {
                ResultsView(viewModel: viewModel)
            }
            .onTapGesture {
                isAnyFieldFocused = false
            }
        }
    }
}

struct ResultsView: View {
    let viewModel: CalculatorViewModel
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        NavigationView {
            let results = viewModel.calculateComparison()
            
            ScrollView {
                VStack(spacing: 20) {
                    Text("Results")
                        .font(.title)
                        .padding()
                    
                    VStack(alignment: .leading, spacing: 10) {
                        Text("Loan Details")
                            .font(.headline)
                        
                        ForEach(viewModel.loans) { loan in
                            VStack(alignment: .leading) {
                                Text(loan.name).fontWeight(.bold)
                                Text("Type: \(loan.selectedLoanType.rawValue)")
                                Text("Term: \(loan.loanTerm) years")
                                
                                let params = viewModel.createLoanParameters(from: loan)
                                let (cost, interest) = viewModel.calculateLoanCost(loanType: loan.selectedLoanType, params: params)
                                
                                Text("Total Cost: $\(String(format: "%.2f", cost))")
                                Text("Interest Paid: $\(String(format: "%.2f", interest))")
                            }
                            .padding(.vertical, 5)
                            
                            if loan.id != viewModel.loans.last?.id {
                                Divider()
                            }
                        }
                        
                        Text("Total Loan Cost: $\(String(format: "%.2f", results.loanCost))")
                            .font(.title3)
                        Text("Total Interest Cost: $\(String(format: "%.2f", results.loanTotalInterest))")
                            .foregroundColor(.red)
                    }
                    .padding()
                    .background(Color(.systemGray6))
                    .cornerRadius(10)
                    .padding(.horizontal)
                    
                    VStack(alignment: .leading, spacing: 10) {
                        Text("Investment Details")
                            .font(.headline)
                        
                        ForEach(viewModel.investments) { investment in
                            VStack(alignment: .leading) {
                                Text(investment.name).fontWeight(.bold)
                                Text("Term: \(investment.investmentTerm) years")
                                Text("Return: \(investment.investmentReturn)%")
                                Text("Monthly Contribution: $\(investment.monthlyContribution)")
                                
                                let value = viewModel.calculateInvestmentValue(for: investment)
                                let principal = Double(investment.investmentAmount) ?? 0
                                let monthlyContribution = Double(investment.monthlyContribution) ?? 0
                                let termInYears = Double(investment.investmentTerm) ?? 0
                                let totalContributions = principal + (monthlyContribution * termInYears * 12)
                                let profit = value - totalContributions
                                
                                Text("Total Invested: $\(String(format: "%.2f", totalContributions))")
                                Text("Final Value: $\(String(format: "%.2f", value))")
                                Text("Profit: $\(String(format: "%.2f", profit))")
                                    .foregroundColor(.green)
                            }
                            .padding(.vertical, 5)
                            
                            if investment.id != viewModel.investments.last?.id {
                                Divider()
                            }
                        }
                        
                        Text("Total Investment Value: $\(String(format: "%.2f", results.investmentValue))")
                            .font(.title3)
                        Text("Total Investment Profit: $\(String(format: "%.2f", results.investmentProfit))")
                            .foregroundColor(.green)
                    }
                    .padding()
                    .background(Color(.systemGray6))
                    .cornerRadius(10)
                    .padding(.horizontal)
                    
                    VStack(alignment: .leading, spacing: 10) {
                        Text("Summary")
                            .font(.headline)
                        
                        HStack {
                            VStack(alignment: .leading) {
                                Text("Investment Profit:")
                                Text("$\(String(format: "%.2f", results.investmentProfit))")
                                    .foregroundColor(.green)
                            }
                            
                            Spacer()
                            
                            VStack(alignment: .trailing) {
                                Text("Loan Interest:")
                                Text("$\(String(format: "%.2f", results.loanTotalInterest))")
                                    .foregroundColor(.red)
                            }
                        }
                        
                        Divider()
                        
                        Text("Net Profit/Loss:")
                            .font(.subheadline)
                        Text("$\(String(format: "%.2f", results.netProfitLoss))")
                            .font(.title2)
                            .foregroundColor(results.netProfitLoss > 0 ? .green : .red)
                        
                        if results.netProfitLoss > 0 {
                            Text("Your investment profits exceed your loan interest costs by $\(String(format: "%.2f", results.netProfitLoss)).")
                                .foregroundColor(.green)
                        } else {
                            Text("Your loan interest costs exceed your investment profits by $\(String(format: "%.2f", abs(results.netProfitLoss))).")
                                .foregroundColor(.red)
                        }
                    }
                    .padding()
                    .background(Color(.systemGray6))
                    .cornerRadius(10)
                    .padding(.horizontal)
                    
                    Button("Close") {
                        dismiss()
                    }
                    .buttonStyle(.borderedProminent)
                    .padding()
                }
                .padding(.bottom, 20)
            }
            .navigationBarTitle("Results", displayMode: .inline)
        }
    }
}

@main
struct LoanVsInvestmentApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
