***HOOKS Methode

const cds = require('@sap/cds');

module.exports = cds.service.impl(async function () {

    const { Products } = this.entities;

    // =========================
    // BEFORE CREATE
    // =========================
    this.before('CREATE', Products, (req) => {

        const data = req.data;

        // Validation: ID mandatory
        if (!data.ID) {
            req.error(400, 'ID is required');
        }

        // Validation: Price must be positive
        if (data.Price <= 0) {
            req.error(400, 'Price must be greater than 0');
        }

        // Auto trim name
        if (data.Name) {
            data.Name = data.Name.trim();
        }

    });

    // =========================
    // BEFORE UPDATE
    // =========================
    this.before('UPDATE', Products, (req) => {

        const data = req.data;

        if (data.Price && data.Price <= 0) {
            req.error(400, 'Price must be greater than 0');
        }

    });

    // =========================
    // AFTER READ
    // =========================
    this.on('READ', Products, async  (req, next) => {
        const data = await next();
        if (Array.isArray(data)) {
            data.forEach(product => {
                product.Description = product.Description + " (Demo)";
            });
        }

        return data;

    });

    // =========================
    // BEFORE DELETE
    // =========================
    this.before('DELETE', Products, async (req) => {

        const { ID } = req.data;

        console.log(`Deleting product: ${ID}`);

    });

});






***CAP backend service logic file defination starting


const cds = require('@sap/cds')

class SalesService extends cds.ApplicationService {

    init(){

        // here we can write our hook methods.
        return super.init();
    }
}

module.exports = {SalesService};