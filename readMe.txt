import { Component, OnInit } from '@angular/core';
import { WorkItemService } from '../work-item.service';
import { jsPDF } from 'jspdf';

@Component({
  selector: 'app-work-item',
  templateUrl: './work-item.component.html',
  styleUrls: ['./work-item.component.css'],
})
export class WorkItemComponent implements OnInit {
  cosmicId: string = ''; // Cosmic ID entered by the user
  selectedWorkItem: any = null; // Store the selected work item
  workItems: any[] = []; // Array to hold all work items fetched from JSON
  showModal: boolean = false; // Flag to control modal display

  constructor(private workItemService: WorkItemService) {}

  ngOnInit() {
    // Fetch all work items when the component initializes
    this.workItemService.getAllWorkItems().subscribe((data) => {
      this.workItems = data.cosmicData; // Store the fetched cosmic data
    });
  }

  // Preview work item details based on entered cosmicId
  preview() {
    this.selectedWorkItem = this.workItemService.getDetailsByCosmicId(
      this.cosmicId,
      this.workItems
    );
    if (this.selectedWorkItem) {
      this.showModal = true; // Open the modal if work item found
    } else {
      alert('CosmicId not found!'); // Handle case if cosmicId is not found
    }
  }

  // Close the modal
  closeModal() {
    this.showModal = false;
  }

  // Function to draw borders around sections in PDF
  drawBorder(
    doc: jsPDF,
    startX: number,
    startY: number,
    endX: number,
    endY: number
  ) {
    doc.setLineWidth(0.5);
    doc.rect(startX, startY, endX - startX, endY - startY); // Draw rectangle for border
  }

  // Function to check if a new page is needed
  checkPageOverflow(
    doc: jsPDF,
    startY: number,
    rowsNeeded: number = 1
  ): number {
    const pageHeight = doc.internal.pageSize.height; // Get the height of the page
    const spaceNeeded = rowsNeeded * 20; // Increase space for row height
    if (startY + spaceNeeded > pageHeight) {
      // Check if space is left for the next rows
      doc.addPage(); // Add a new page
      return 10; // Reset startY to a small margin
    }
    return startY; // Return the current startY if no new page is needed
  }

  // Download the selected data as a PDF using jsPDF
  downloadPDF() {
    if (!this.selectedWorkItem) {
      alert('Please select a work item first.');
      return;
    }

    const doc = new jsPDF();
    let startY = 10;

    // Title
    doc.setFontSize(16);
    doc.text('Details for Cosmic ID: ' + this.cosmicId, 10, startY);
    startY += 10;

    // Function to add spacing between sections (reduced to avoid extra gap)
    const addSpacing = (spacing: number) => {
      startY += spacing;
    };

    // 1. Cosmic Work Item Table
    doc.setFontSize(12);
    doc.text('Cosmic Work Item:', 10, startY);
    startY += 6; // Reduced spacing between section headers and table

    const workItemDetails = [
      ['Item ID:', this.selectedWorkItem.cosmicId],
      ['Parent Case ID:', this.selectedWorkItem.parentCaseId],
      ['Sharing Category:', this.selectedWorkItem.summary[0].sharingCategory],
      ['Sending Bank:', this.selectedWorkItem.summary[0].sendingBank.label],
      ['Business Unit:', this.selectedWorkItem.businessUnit],
    ];

    startY = this.addTable(doc, workItemDetails, startY);
    startY += 8; // Reduced spacing after table

    // 2. Cosmic Case Item Details Table
    doc.setFontSize(12);
    doc.text('Cosmic Case Item Details:', 10, startY);
    startY += 6;

    const cosmicCaseDetails = [
      ['Item ID:', this.selectedWorkItem.cosmicId],
      ['Receiving Bank:', this.selectedWorkItem.summary[0].receivingBank.label],
      ['Focus Name:', this.selectedWorkItem.focusName],
      [
        'Red Flags Identified:',
        this.selectedWorkItem.summary[0].redFlags.length.toString(),
      ],
      ['Focus ID:', this.selectedWorkItem.focusIdentifier],
      ['Business Unit:', this.selectedWorkItem.businessUnit],
      ['Sharing Category:', this.selectedWorkItem.summary[0].sharingCategory],
      ['Sending Bank:', this.selectedWorkItem.summary[0].sendingBank.label],
    ];

    startY = this.addTable(doc, cosmicCaseDetails, startY);
    startY += 8;

    // 3. Summary Information Table
    doc.setFontSize(12);
    doc.text('Summary Information:', 10, startY);
    startY += 6;

    const summaryDetails = [
      [
        'Type of Risk:',
        this.selectedWorkItem.summary[0].redFlags[0].redFlagType,
      ],
      ['Sharing Category:', this.selectedWorkItem.summary[0].sharingCategory],
      [
        'Red Flags Identified:',
        this.selectedWorkItem.summary[0].redFlags.length.toString(),
      ],
      ['Sending Bank:', this.selectedWorkItem.summary[0].sendingBank.label],
      [
        'Receiving Bank:',
        this.selectedWorkItem.summary[0].receivingBank.label,
      ],
      [
        'Existing Ticket Reason:',
        this.selectedWorkItem.summary[0].existingTicketReason,
      ],
    ];

    startY = this.addTable(doc, summaryDetails, startY);
    startY += 8;

    // 4. List of Red Flags Identified Table
    doc.setFontSize(12);
    doc.text('List of Red Flags Identified:', 10, startY);
    startY += 6;

    const redFlagDetails = this.selectedWorkItem.summary[0].redFlags
      .map((redFlag: any) => [
        ['Type of Risk:', redFlag.redFlagType],
        ['Description:', redFlag.description],
      ])
      .flat();

    startY = this.addTable(doc, redFlagDetails, startY);
    startY += 8;

    // 5. Entity and Account Information Table
    doc.setFontSize(12);
    doc.text('Entity and Account Information:', 10, startY);
    startY += 6;

    const entityAccountDetails = [
      ['Category:', this.selectedWorkItem.entityInformation[0].category],
      ['Entity Type:', this.selectedWorkItem.entityInformation[0].type],
      ['Entity Name:', this.selectedWorkItem.entityInformation[0].name],
      [
        'Date of Birth/Incorporation:',
        this.selectedWorkItem.entityInformation[0].dateofBirthIncorporatedDate,
      ],
      [
        'Country of Incorporation:',
        this.selectedWorkItem.entityInformation[0].countryOfIncorporation,
      ],
      [
        'Corporate Registry Number:',
        this.selectedWorkItem.entityInformation[0].corporateRegistryNumber,
      ],
      [
        'Included in Alert:',
        this.selectedWorkItem.entityInformation[0].includedInAlert,
      ],
      [
        'Alert Justification:',
        this.selectedWorkItem.entityInformation[0].alertJustification,
      ],
    ];

    startY = this.addTable(doc, entityAccountDetails, startY);
    startY += 8;

    // 6. Nationality Table
    doc.setFontSize(12);
    doc.text('Nationality:', 10, startY);
    startY += 6;
    const nationalityDetails = [
      ['Country Code:', this.selectedWorkItem.entityInformation[0].countryOfIncorporation],
      ['Identification Type:', this.selectedWorkItem.entityInformation[0].alertJustification],
    ];

    startY = this.addTable(doc, nationalityDetails, startY);
    startY += 8; // Update startY

    // 7. Telephone Number Table
    doc.setFontSize(12);
    doc.text('Telephone Number:', 10, startY);
    startY += 6;
    const telephoneDetails = [
      ['Telephone Type:', this.selectedWorkItem.entityInformation[0].telephoneNumbers[0].telephoneType.label],
      ['Country Code:', this.selectedWorkItem.entityInformation[0].telephoneNumbers[0].countryCode],
      ['Area Code:', this.selectedWorkItem.entityInformation[0].telephoneNumbers[0].areaCode],
      ['Number:', this.selectedWorkItem.entityInformation[0].telephoneNumbers[0].number],
    ];

    startY = this.addTable(doc, telephoneDetails, startY);
    startY += 8; // Update startY

    // 8. Address Table
    doc.setFontSize(12);
    doc.text('Address:', 10, startY);
    startY += 6;
    const addressDetails = [
      ['Structured:', this.selectedWorkItem.entityInformation[0].addressess[0].isStructured],
      ['Address Type:', this.selectedWorkItem.entityInformation[0].addressess[0].addressType.label],
      ['Country Code:', this.selectedWorkItem.entityInformation[0].addressess[0].countryCode],
      ['Address Line 1:', this.selectedWorkItem.entityInformation[0].addressess[0].addressLine1],
      ['Address Line 2:', this.selectedWorkItem.entityInformation[0].addressess[0].addressLine2],
      ['Address Line 3:', this.selectedWorkItem.entityInformation[0].addressess[0].addressLine3],
      ['Address Line 4:', this.selectedWorkItem.entityInformation[0].addressess[0].addressLine4],
      ['Address Line 5:', this.selectedWorkItem.entityInformation[0].addressess[0].addressLine5],
    ];

    startY = this.addTable(doc, addressDetails, startY);
    startY += 8; // Update startY

     // 9. Entity Accounts Table
     doc.setFontSize(12);
     doc.text('Entity Accounts:', 10, startY);
     startY += 6;
     const entityAccountsDetails = [
       ['Account Number:', this.selectedWorkItem.accounts[0].accountNumber],
       ['Account Type:', this.selectedWorkItem.accounts[0].accountType.label],
       ['Account Open Date:', this.selectedWorkItem.accounts[0].accountOpenDate],
       ['Account Status:', this.selectedWorkItem.accounts[0].accountStatus.label],
     ];

     startY = this.addTable(doc, entityAccountsDetails, startY);
    startY += 8; // Update startY

    // 6. Transactions Table
    doc.setFontSize(12);
    doc.text('Transactions:', 10, startY);
    startY += 6;

    const transactionDetails = [
      ['Transaction Reference ID:', this.selectedWorkItem.transaction[0].transactionReferenceId],
      ['Transaction Date:', this.selectedWorkItem.transaction[0].transactionDate],
      ['Originator Name:', this.selectedWorkItem.transaction[0].originatorName],
      ['Originating Account No:', this.selectedWorkItem.transaction[0].originatingAccountNo],
      ['Originating Bank Code:', this.selectedWorkItem.transaction[0].originatingBankCode.label],
      ['Amount:', this.selectedWorkItem.transaction[0].amount],
      ['Currency Code:', this.selectedWorkItem.transaction[0].currencyCode.label],
      ['SGD-equivalent:', this.selectedWorkItem.transaction[0].sgdEquivalentAmount],
      ['Beneficiary Name:', this.selectedWorkItem.transaction[0].beneficiaryName],
      ['Beneficiary Account No:', this.selectedWorkItem.transaction[0].beneficiaryAccountNo],
      ['Beneficiary Bank Code:', this.selectedWorkItem.transaction[0].beneficiaryBankCode.label],
      ['Summary of Observation:', this.selectedWorkItem.summary[0].summaryDescription],
    ];

    startY = this.addTable(doc, transactionDetails, startY);
    startY += 8; // Update startY

    // 7. Beneficiaries Table
    doc.setFontSize(12);
    doc.text('Beneficiaries:', 10, startY);
    startY += 6;

    const beneficiariesDetails = [
      ['Beneficiary Account No:', this.selectedWorkItem.transaction[0].beneficiaryAccountNo],
      ['Beneficiary Name:', this.selectedWorkItem.transaction[0].beneficiaryName],
      ['Transaction Date:', this.selectedWorkItem.transaction[0].transactionDate],
      ['Amount:', this.selectedWorkItem.transaction[0].amount],
      ['Currency Code:', this.selectedWorkItem.transaction[0].currencyCode.label],
      ['SGD-equivalent:', this.selectedWorkItem.transaction[0].sgdEquivalentAmount],
      ['Beneficiary Bank Code:', this.selectedWorkItem.transaction[0].beneficiaryBankCode.label],
    ];
      

    startY = this.addTable(doc, beneficiariesDetails, startY);
    startY += 8; // Update startY

    // Save the PDF with Cosmic ID in the filename
    doc.save(`Cosmic_${this.cosmicId}.pdf`);
  }

  // Function to add a custom table with four cells per row without using autoTable
  addTable(doc: jsPDF, data: any[][], startY: number): number {
    const rowHeight = 14; // Increased row height
    const colWidth = 48; // Column width for each column
    const startX = 10;   // Starting X position
    let currentY = startY;

    // Loop through the data and create cells with 4 columns per row
    for (let i = 0; i < data.length; i += 4) {
      let currentX = startX;

      for (let j = 0; j < 4; j++) {
        // Check if there's a value for the current cell (it might be missing in the last row)
        if (i + j < data.length) {
          const cell = data[i + j];
          const label = cell[0];
          const value = cell[1];

          // Draw cell border and add text for the current cell
          this.drawBorder(doc, currentX, currentY, currentX + colWidth, currentY + rowHeight);
          doc.setFontSize(10);
          doc.text(`${label}`, currentX + 2, currentY + 6);
          
          // Adjust text wrapping if it exceeds the column width
          const wrappedValue = doc.splitTextToSize(`${value}`, colWidth - 4);
          doc.text(wrappedValue, currentX + 2, currentY + 12);

          // Move to the next column
          currentX += colWidth;
        }
      }

      // Move to the next row after 4 columns
      currentY += rowHeight;

      // Check for page overflow and add a new page if necessary
      currentY = this.checkPageOverflow(doc, currentY);
    }

    // Return the updated currentY position for further table additions
    return currentY;
  }
}