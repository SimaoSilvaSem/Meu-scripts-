# Meu-scripts-
// ==UserScript==
// @name         Script Enviar Ataques - Com Auto-Offset
// @author       DuMal Team!
// @version      2.0
// @description   Versão original com auto-detecção de offset e interface melhorada
// @include      https://pt*screen=place&try=confirm*
// @include      https://pt*screen=place&mode=command&try=confirm*
// @run-at       document-start
// @grant        GM_setValue
// @grant        GM_getValue
// ==/UserScript==

(function() {
    'use strict';

    // ============================================
    // CÓDIGO BASE ORIGINAL COM AUTO-OFFSET
    // ============================================

    let timeoutId = null;

    // Função para medir ping e calcular offset
    async function measureOffset() {
        const btn = $('#CSautoOffset');
        const originalText = btn.text();
        btn.text('📡 Medindo...').prop('disabled', true);

        try {
            // Faz 5 medições para maior precisão
            const pings = [];
            for (let i = 0; i < 5; i++) {
                const start = performance.now();
                await new Promise((resolve) => {
                    const img = new Image();
                    img.onload = img.onerror = () => {
                        pings.push(performance.now() - start);
                        resolve();
                    };
                    img.src = window.location.origin + '/favicon.ico?_=' + Date.now() + i;
                });
                await new Promise(r => setTimeout(r, 100));
            }

            // Remove outlier (maior e menor)
            pings.sort((a, b) => a - b);
            let avgPing = pings[2]; // mediana das 5 medições
            if (pings.length >= 3) {
                avgPing = (pings[1] + pings[2] + pings[3]) / 3;
            }

            // Offset recomendado = metade do ping
            const recommended = Math.floor(avgPing / 2);

            // Aplica o offset
            $('#CSoffset').val(recommended);
            CommandSender.offset = recommended;
            GM_setValue('CS.offset', recommended);

            btn.text(`✅ ${recommended}ms`);
            setTimeout(() => {
                btn.text(originalText).prop('disabled', false);
            }, 1500);

            console.log(`📊 Ping médio: ${Math.floor(avgPing)}ms | Offset recomendado: ${recommended}ms`);
        } catch(e) {
            btn.text('❌ Falha');
            setTimeout(() => {
                btn.text(originalText).prop('disabled', false);
            }, 1500);
            console.error('Erro ao medir offset:', e);
        }
    }

    const CommandSender = {
        confirmButton: null,
        duration: null,
        dateNow: null,
        offset: null,

        init: function () {
            // Adiciona os campos com botão auto-offset
            $($('#command-data-form')['find']('tbody')[0])['append'](
                '<tr><td style="font-weight:bold;">📅 Chegada:</td><td>' +
                '<input type="datetime-local" id="CStime" step=".001" style="font-size:10pt;font-family:monospace;padding:6px;border-radius:6px;border:1px solid #00ffcc;background:#0a0a0f;color:#00ffcc;">' +
                '</td></tr>' +
                '<tr><td style="font-weight:bold;">⚙️ Offset (ms):</td><td>' +
                '<input type="number" id="CSoffset" style="width:100px;font-size:10pt;padding:6px;border-radius:6px;border:1px solid #00ffcc;background:#0a0a0f;color:#00ffcc;"> ' +
                '<button type="button" id="CSautoOffset" class="btn" style="background:#2a2a3a;color:#00ffcc;border:none;padding:6px 12px;border-radius:6px;cursor:pointer;margin-right:8px;">🎯 Auto</button>' +
                '<button type="button" id="CSbutton" class="btn" style="background:linear-gradient(135deg,#00ffcc,#00ccff);color:#000;font-weight:bold;border:none;padding:6px 16px;border-radius:6px;cursor:pointer;">✅ Agendar</button>' +
                '</td></tr>'
            );

            this['confirmButton'] = $('#troop_confirm_submit');
            this['duration'] = $('#command-data-form')['find']('td:contains("Duração:")')['next']()['text']()['split'](':')['map'](Number);
            this['offset'] = GM_getValue('CS.offset', -250);
            this['dateNow'] = this['convertToInput'](new Date());

            $('#CSoffset')['val'](this['offset']);
            $('#CStime')['val'](this['dateNow']);

... (124 linhas)
