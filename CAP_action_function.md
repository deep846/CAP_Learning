1. CAP action Functions


               // -------- FUNCTIONS (GET semantics in OData, but called via bindContext in UI5 V4) --------
        onErrorFunction: async function () {
            const oModel = this._getModel();
            const oFn = oModel.bindContext("/errorFuction(...)"); // keep exact name
            try {
                await oFn.execute();
                const res = await oFn.requestObject(); // { ok, message, echo }
                if (res?.ok) {
                    MessageBox.success(`errorFuction OK: ${res.message} | echo=${res.echo ?? ""}`);
                } else {
                    // if backend returns ok=false but no exception
                    MessageBox.warning(res?.message || "errorFuction returned ok=false");
                }
            } catch (e) {
                console.error("errorFuction failed:", e);
                MessageBox.error(e?.message || "errorFuction failed");
            }
        },

        onSuccessFunction: async function () {
            const oModel = this._getModel();
            const oFn = oModel.bindContext("/successFunction(...)");
            try {
                await oFn.execute();
                const res = await oFn.requestObject(); // { ok, message, echo }
                if (res?.ok) {
                    MessageBox.success(`successFunction OK: ${res.message} | echo=${res.echo ?? ""}`);
                } else {
                    MessageBox.warning(res?.message || "successFunction returned ok=false");
                }
            } catch (e) {
                console.error("successFunction failed:", e);
                MessageBox.error(e?.message || "successFunction failed");
            }
        },

        // -------- ACTIONS (POST semantics) --------
        onErrorAction: async function () {
            const oModel = this._getModel();
            const oAction = oModel.bindContext("/errorAction(...)");
            try {
                await oAction.execute();
                const res = await oAction.requestObject(); // may be undefined if no return
                if (res && (res.message || res.status)) {
                    MessageBox.success(`errorAction response: ${res.status ?? "N/A"} - ${res.message ?? ""}`);
                } else {
                    MessageToast.show("errorAction executed");
                }
            } catch (e) {
                console.error("errorAction failed:", e);
                MessageBox.error(e?.message || "errorAction failed");
            }
        },

        onSuccessAction: async function () {
            const oModel = this._getModel();
            const oAction = oModel.bindContext("/successAction(...)");
            try {
                await oAction.execute();
                const res = await oAction.requestObject(); // may be undefined if no return
                if (res && (res.message || res.status)) {
                    MessageBox.success(`successAction response: ${res.status ?? "OK"} - ${res.message ?? ""}`);
                } else {
                    MessageBox.success("successAction executed successfully");
                }
            } catch (e) {
                console.error("successAction failed:", e);
                MessageBox.error(e?.message || "successAction failed");
            }
        },


        
        onShowUserInfo: async function () {
            try {
                const oModel = this.getView().getModel();   // OData V4 model

                const oFn = oModel.bindContext("/me(...)");
                await oFn.execute();

                const me = await oFn.requestObject();

                MessageBox.information(
                    `Logged-in User Details:\n\n` +
                    `User ID : ${me.id}\n` +
                    `Email   : ${me.email}\n` +
                    `Roles   : ${me.roles.join(", ")}`
                );

            } catch (e) {
                console.error("Failed to load user info", e);
                MessageBox.error("Unable to retrieve user info");
            }
        },











        //  Function with params + return → /computeScore(...)

        async onComputeScore() {
            const oModel = this.getView().getModel();
            const oFn = oModel.bindContext("/computeScore(...)");
            oFn.setParameter("name", "Deep");
            oFn.setParameter("age", 10);

            try {
                await oFn.execute();
                const res = await oFn.requestObject(); // { ok, message, score }
                sap.m.MessageToast.show(`${res.message} (score=${res.score})`);
            } catch (e) {
                sap.m.MessageBox.error(e?.message || "computeScore failed");
            }
        },

        // Function with params, no return → /checkEligibility(...)

        async onCheckEligibility() {
            const oModel = this.getView().getModel();
            const oFn = oModel.bindContext("/checkEligibility(...)");
            oFn.setParameter("age", 23);
            oFn.setParameter("enrolled", true);

            try {
                await oFn.execute();
                await oFn.requestObject(); // likely {}
                sap.m.MessageToast.show("Eligibility check completed");
            } catch (e) {
                sap.m.MessageBox.error(e?.message || "checkEligibility failed");
            }
        },

        // Action with params + return → /upsertStudent(...)
        async onUpsertStudent() {
            const oModel = this.getView().getModel();
            const oAction = oModel.bindContext("/upsertStudent(...)");
            oAction.setParameter("student", {
                name: "name",
                email: "deep@gmail.com",
                enrolled: true
            });

            try {
                await oAction.execute();
                const res = await oAction.requestObject(); // { id, status, message }
                sap.m.MessageToast.show(`${res.status}: ${res.message}`);
            } catch (e) {
                sap.m.MessageBox.error(e?.message || "upsertStudent failed");
            }
        },


        // 4) Action with params, no return → /sendNotification(...)

        async onSendNotification() {
            const oModel = this.getView().getModel();
            const oAction = oModel.bindContext("/sendNotification(...)");
            oAction.setParameter("targetEmail", "Deep");
            // oAction.setParameter("subject", "English");
            oAction.setParameter("body", "asdlkfjasdf");

            try {
                await oAction.execute();
                await oAction.requestObject(); // undefined / empty
                sap.m.MessageToast.show("Notification sent");
            } catch (e) {
                sap.m.MessageBox.error(e?.message || "sendNotification failed");
            }
        }